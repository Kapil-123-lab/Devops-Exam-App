pipeline {
agent any


environment {
    IMAGE_NAME = "kapilkanaujiya/kapil-123-lab:latest"
}

stages {

    stage('Git Checkout') {
        steps {
            git branch: 'main',
            url: 'https://github.com/Kapil-123-lab/Devops-Exam-App.git'
        }
    }

    stage('Verify Docker & Docker Compose') {
        steps {
            bat 'docker --version'
            bat 'docker compose version'
        }
    }

    stage('Build Docker Image') {
        steps {
            dir('backend') {
                bat 'docker build -t %IMAGE_NAME% .'
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            script {
                withDockerRegistry([credentialsId: 'docker-creds', url: '']) {
                    bat 'docker push %IMAGE_NAME%'
                }
            }
        }
    }

    stage('Deploy Application') {
        steps {
            bat 'docker compose down'
            bat 'docker compose up -d'
        }
    }

    stage('Verify Deployment') {
        steps {
            bat 'docker ps'
        }
    }
}

post {
    success {
        echo 'Pipeline Success!'
    }

    failure {
        echo 'Pipeline Failed!'
        bat 'docker compose logs --tail=50'
    }
}

}
