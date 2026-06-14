pipeline {
    agent any
    environment {
        IMAGE_NAME = "rachanayelmame8/portfolio-generator"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        PROD_IP    = "3.108.54.110"
    }
    stages {

        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jenkins-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Production') {
            steps {
                sshagent(credentials: ['ssh-with-private-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@$PROD_IP "
                            docker pull $IMAGE_NAME:latest
                            docker rm -f portfolio-app || true
                            docker run -d \
                                --name portfolio-app \
                                --restart always \
                                -p 5000:5000 \
                                $IMAGE_NAME:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ Deployed - Build ${BUILD_NUMBER}" }
        failure { echo "❌ Failed - Check logs" }
    }
}
