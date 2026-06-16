   pipeline {
    agent any

    environment {
        IMAGE_NAME = "your-dockerhub-username/my-python-app"
        SERVER_IP  = "<EC2-2-PRIVATE-IP>"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-deploy-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                    docker pull $IMAGE_NAME:latest
                    docker stop my-python-app || true
                    docker rm my-python-app || true
                    docker run -d --name my-python-app -p 8000:8000 $IMAGE_NAME:latest
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }

        failure {
            echo 'Deployment Failed!'
        }
    }
}        
   
