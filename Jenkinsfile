pipeline {
agent any


environment {
    DOCKER_IMAGE = "kapilkanaujiya/kapil-123-lab:latest"
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
            sh '''
            docker --version

            docker compose version || {
                echo "Docker Compose not installed"
                exit 1
            }
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            dir('backend') {
                sh '''
                docker build -t ${DOCKER_IMAGE} .
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            script {
                withDockerRegistry(credentialsId: 'docker-creds') {

                    sh '''
                    docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }
    }

    stage('Deploy Application') {
        steps {
            sh '''
            docker compose down --remove-orphans || true

            docker pull ${DOCKER_IMAGE} || true

            docker compose up -d
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            docker compose ps -a

            curl -I http://localhost:5000 || true
            '''
        }
    }
}

post {

    success {
        echo 'Deployment Successful!'
    }

    failure {
        echo 'Pipeline Failed!'

        sh '''
        docker compose logs --tail=50 || true
        '''
    }
}

}
