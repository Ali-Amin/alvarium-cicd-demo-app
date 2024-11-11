@Library('alvarium-pipelines') _

pipeline {
    agent any
    tools {
	go 'go-1.21'
    }
    environment {
	GO121MODULE = 'on'
	TAG = "${GIT_COMMIT}"
	DOCKERHUB_CREDENTIALS = credentials("dockerhub_id")
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
		sh 'go build -o cmd/transitor/transitor-demo ./cmd/transitor'
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
		    sh 'docker build -t alimamin/transitor-demo --build-arg TAG=${TAG} -f Dockerfile.transitor .'
		    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'

		    sh 'docker push alimamin/transitor-demo:latest'
		}
	    }
	}
    }
}
