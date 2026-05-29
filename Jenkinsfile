pipeline {
    agent any

    environment {
        // Automatically tags your image with the active Jenkins Build Number
        IMAGE_NAME = "kapilkanaujiya/kapil-123-lab:${BUILD_NUMBER}"
        EC2_USER   = "ubuntu"
        EC2_IP     = "3.108.249.247"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kapil-123-lab/Devops-Exam-App.git'
            }
        }

        stage('Verify Environment') {
            steps {
                bat 'docker --version'
                bat 'docker compose version'
                echo "======================================"
                echo "EXECUTING BUILD NUMBER: ${env.BUILD_NUMBER}"
                echo "======================================"
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    bat "docker build -t %IMAGE_NAME% ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker', url: '']) {
                        bat "docker push %IMAGE_NAME%"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (Minikube on EC2)') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'PEM_FILE')]) {
                    bat """
                    :: Reset permissions first to avoid conflicts
                    icacls "%PEM_FILE%" /reset
                    
                    :: Disable inheritance cleanly
                    icacls "%PEM_FILE%" /inheritance:r /c /q
                    
                    :: Grant explicit read access to the local SYSTEM account (S-1-5-18) and Administrators group
                    icacls "%PEM_FILE%" /grant *S-1-5-18:R /c /q
                    icacls "%PEM_FILE%" /grant *S-1-5-32-544:R /c /q
                    
                    :: Run the SSH deployment command
                    ssh -o StrictHostKeyChecking=no -i "%PEM_FILE%" ${EC2_USER}@${EC2_IP} "cd ~/Devops-Exam-App && git pull origin main && sed -i 's|image: kapilkanaujiya/kapil-123-lab:.*|image: ${IMAGE_NAME}|g' k8s/deployment.yaml && kubectl apply -f k8s/ && kubectl rollout status deployment/flask-app --timeout=60s"
                    """
                }
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'PEM_FILE')]) {
                    bat """
                    icacls "%PEM_FILE%" /reset
                    icacls "%PEM_FILE%" /inheritance:r /c /q
                    icacls "%PEM_FILE%" /grant *S-1-5-18:R /c /q
                    icacls "%PEM_FILE%" /grant *S-1-5-32-544:R /c /q
                    
                    ssh -o StrictHostKeyChecking=no -i "%PEM_FILE%" ${EC2_USER}@${EC2_IP} "echo '=== CURRENT RUNNING PODS ===' && kubectl get pods && echo '=== ACTIVE SERVICES ===' && kubectl get svc"
                    """
                }
            }
        }
    } // <--- Added: This properly closes the 'stages' block

    post {
        success {
            echo '======================================'
            echo '🎉 Pipeline Finished Successfully!'
            echo '======================================'
        }
        failure {
            echo '======================================'
            echo '❌ Pipeline Execution Failed!'
            echo '======================================'
        }
    }
}
