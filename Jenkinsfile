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
                echo "Stopping old containers..."
                docker compose down --remove-orphans || true

                echo "Pulling latest image..."
                docker pull ${DOCKER_IMAGE} || true

                echo "Starting containers..."
                docker compose up -d

                echo "Waiting for MySQL..."

                timeout 120s bash -c '
                until docker compose exec -T mysql mysqladmin ping -uroot -prootpass --silent
                do
                    echo "MySQL not ready yet..."
                    sleep 5
                done
                '

                echo "Application deployment completed"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "===== RUNNING CONTAINERS ====="
                docker compose ps -a

                echo "===== APPLICATION TEST ====="
                curl -I http://localhost:5000 || true

                echo "===== DOCKER IMAGES ====="
                docker images | grep kapil-123-lab || true
                '''
            }
        }
    }

    post {

        success {
            echo '🚀 Deployment Successful!'
        }

        failure {
            echo '❌ Pipeline Failed!'

            sh '''
            echo "===== DOCKER COMPOSE LOGS ====="
            docker compose logs --tail=50 || true
            '''
        }

        always {
            sh '''
            echo "===== FINAL CONTAINER STATUS ====="
            docker compose ps -a || true
            '''
        }
    }
}
```
