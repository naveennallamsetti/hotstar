pipeline{
    agent any
    stages {
        stage (' git checkout ') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'naveencred', url: 'https://github.com/naveennallamsetti/hotstar.git']])
            }
        }
        stage ('STAGE VALIDATE') {
            steps {
                sh 'mvn validate'
            }
        }
        stage ('STAGE COMPILE') {
            steps {
                sh 'mvn compile'
            }
        }
        stage ('STAGE TEST') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('STAGE PACKAGE') {
            steps {
                sh 'mvn package'
            }
        }
      stage('Docker Image Build') {
            steps {
                sh 'docker rmi project1 || true'
                sh 'docker build -t project3 .'
            }
        }
        stage('Deploy Container') {
            steps {
                sh 'docker rm project1-container -f || true'
                sh 'docker run -d -p 8093:8080 --name project1-container project1'
            }
        }
    }
}
