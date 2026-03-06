pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveennallamsetti/docker-my-images"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {

        stage('Git Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        credentialsId: 'naveencred',
                        url: 'https://github.com/naveennallamsetti/hotstar.git'
                    ]]
                )
            }
        }
        stage('Validate') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'naveendoc-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }
        stage('Docker Push') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                docker push ${DOCKER_IMAGE}:latest
                """
            }
        }
        stage('Deploy Container') {
            steps {
                sh """
                docker rm -f project1-container || true
                docker run -d -p 8093:8080 --name project1-container ${DOCKER_IMAGE}:${IMAGE_TAG}
                """
            }
        }
        stage('Cleanup Local Images') {
            steps {
                sh """
                docker image prune -f
                docker container prune -f
                """
            }
        }
    }
}