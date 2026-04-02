pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = "naveennallamsetti/docker-my-images"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONARQUBE_ENV = 'sq'
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        credentialsId: 'naveen-git',
                        url: 'https://github.com/naveennallamsetti/hotstar.git'
                    ]]
                )
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'setting.xml',
                    jdk: 'jdk17',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh 'mvn deploy'
                }
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
                    credentialsId: 'naveen-docker',
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
                docker run -d -p 9000:9000 --name project1-container ${DOCKER_IMAGE}:${IMAGE_TAG}
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
