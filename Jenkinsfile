pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    # Stop and remove only the container for this environment
                    docker rm -f ${IMAGE_NAME} || true
                    # Run the new container
                    docker run -d --name ${IMAGE_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
