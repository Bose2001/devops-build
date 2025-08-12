pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'bose2001'
        DOCKER_IMAGE = 'bose2001/prod'
        DOCKER_HUB_CREDS = 'dockerhub-creds'
        EC2_USER = 'ec2-user'
        EC2_IP = '3.92.244.1'  // Replace with your actual EC2 instance Public IP
        EC2_SSH_KEY = 'ec2-ssh-key'
        CONTAINER_NAME = 'react-app'
        PORT = '80'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE}:latest .
                    """
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent([EC2_SSH_KEY]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            docker pull ${DOCKER_IMAGE}:latest &&
                            docker rm -f ${CONTAINER_NAME} || true &&
                            docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${DOCKER_IMAGE}:latest
                        '
                        """
                    }
                }
            }
        }
    }
}
