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
                withDockerRegistry([credentialsId: 'docker', url: '']) {
                    bat 'docker push %IMAGE_NAME%'
                }
            }
        }
    }

    stage('Deploy Application Locally') {
        steps {
            bat 'docker compose down'
            bat 'docker compose up -d'
        }
    }

    stage('Deploy to EC2') {
    steps {
        bat '''
        icacls "C:\\Users\\Alg gaming\\Downloads\\Devops Exam app.pem" /inheritance:r
        icacls "C:\\Users\\Alg gaming\\Downloads\\Devops Exam app.pem" /grant:r "%USERNAME%:R"

        ssh -o StrictHostKeyChecking=no -i "C:\\Users\\Alg gaming\\Downloads\\Devops Exam app.pem" ubuntu@3.108.249.247 ^
        "docker pull kapilkanaujiya/kapil-123-lab:latest && ^
        docker stop flask_app || true && ^
        docker rm flask_app || true && ^
        docker run -d --name flask_app -p 5000:5000 kapilkanaujiya/kapil-123-lab:latest"
        '''
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
