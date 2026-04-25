pipeline {
    agent none

    environment {
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG = 'v1.0'
        DOCKER_REPO = 'juanjbetancurm/cicd-pipeline'
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                stash name: 'source', includes: '**'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:7.8.0'
                    args '-v /tmp/npm-cache:/root/.npm'
                }
            }
            steps {
                unstash 'source'
                sh 'npm install'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:7.8.0'
                    args '-v /tmp/npm-cache:/root/.npm'
                }
            }
            steps {
                unstash 'source'
                sh 'npm install'
                sh 'CI=true npm test'
            }
        }

        stage('Lint Dockerfile') {
            agent any
            steps {
                unstash 'source'
                sh 'hadolint Dockerfile || true'
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                unstash 'source'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            agent any
            steps {
                script {
                    def vulnerabilities = sh(
                        script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${IMAGE_NAME}:${IMAGE_TAG}",
                        returnStdout: true
                    ).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }

        stage('Push to Docker Hub') {
            agent any
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REPO}:${IMAGE_NAME}-${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:${IMAGE_NAME}-${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                sh """
                    docker rm -f ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Trigger Deploy Pipeline') {
            agent any
            steps {
                script {
                    def deployJob = env.BRANCH_NAME == 'main' ? 'Deploy_to_main' : 'Deploy_to_dev'
                    build job: deployJob, wait: false
                }
            }
        }
    }
}
