pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kapilkanaujiya/kapil-123-lab:latest"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/Kapil-123-lab/Devops-Exam-App.git',
                    branch: 'main'
            }
        }   
        stage('Verify Docker Compose') {
            steps {
                sh '''
                docker compose version || { echo "Docker Compose not available"; exit 1; }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    script {
                        withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                            sh "docker build -t ${DOCKER_IMAGE} ."
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh """
                        docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                docker compose down --remove-orphans || true

                docker compose up -d

                echo "Waiting for MySQL to be ready..."

                timeout 120s bash -c '
                while ! docker compose exec -T mysql mysqladmin ping -uroot -prootpass --silent;
                do
                    sleep 5
                    docker compose logs mysql --tail=5 || true
                done'

                sleep 10
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "=== Container Status ==="
                docker compose ps -a

                echo "=== Testing Flask Endpoint ==="
                curl -I http://localhost:5000 || true
                '''
            }
        }
    }

    post {
        success {
            echo '🚀 Deployment successful!'

            sh '''
            docker compose ps
            docker images | grep kapil-123-lab
            '''
        }

        failure {
            echo '❗ Pipeline failed. Check logs above.'

            sh '''
            echo "=== Error Investigation ==="
            docker compose logs --tail=50 || true
            '''
        }

        always {
            sh '''
            echo "=== Final Logs ==="
            docker compose logs --tail=20 || true
            '''
        }
    }
}
