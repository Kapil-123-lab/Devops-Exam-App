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
            bat '''
            docker --version

            docker compose version
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            dir('backend') {

                bat '''
                docker build -t %DOCKER_IMAGE% .
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            script {

                withDockerRegistry(credentialsId: 'docker-creds') {

                    bat '''
                    docker push %DOCKER_IMAGE%
                    '''
                }
            }
        }
    }

    stage('Deploy Application') {
        steps {

            bat '''
            docker compose down --remove-orphans

            docker pull %DOCKER_IMAGE%

            docker compose up -d
            '''
        }
    }

    stage('Verify Deployment') {
        steps {

            bat '''
            docker compose ps -a

            curl http://localhost:5000
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

        bat '''
        docker compose logs --tail=50
        '''
    }

    always {

        bat '''
        docker compose ps -a
        '''
    }
}


}
