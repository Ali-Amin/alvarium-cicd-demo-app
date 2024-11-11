@Library('alvarium-pipelines') _

pipeline {
    agent any
    tools {
        go 'go-1.21'
    }
    environment {
        GO121MODULE = 'on'
        TAG = "${GIT_COMMIT}"
    }
    stages {
        stage('prep - generate source code checksum') {
            steps {
                sh 'mkdir -p $JENKINS_HOME/jobs/$JOB_NAME/$BUILD_NUMBER/'
                sh '''find . -type f -exec sha256sum {} + |\
                        md5sum |\
                        cut -d" " -f1 \
                        > $JENKINS_HOME/jobs/$JOB_NAME/$BUILD_NUMBER/sc_checksum
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'go build -o cmd/creator/creator-demo ./cmd/creator'
            }
        }

        stage('alvarium - pre-build annotations') {
            steps {
                script {
                    def optionalParams = ['sourceCodeChecksumPath':"${JENKINS_HOME}/jobs/${JOB_NAME}/${BUILD_NUMBER}/sc_checksum"]
                    alvariumCreate(['source-code', 'vulnerability'], optionalParams)
                }
            }
        }

        stage('Dockerize') {
            steps {
                script {
                    // Define the docker image names
		    sh "docker build --build-arg TAG=${TAG} -t creator-demo -f Dockerfile.creator ."
                }
            }
        }
    }
}
