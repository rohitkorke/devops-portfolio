pipeline {
    agent any

    environment {
        IMAGE_NAME = "portfolio-image"
        DOCKERHUB_USER = "rohitkorke"     // <-- apna username dal
        CONTAINER_NAME = "portfolio-container"
        EC2_INSTANCE_ID = "i-02ac3288e6eacc6f9"      // <-- apna EC2 ID dal
        AWS_REGION = "ap-south-1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rohitkorke/devops-portfolio.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'docker',
                    passwordVariable: 'docker'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Tag Image') {
            steps {
                sh "docker tag ${IMAGE_NAME} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EC2 via SSM') {
            steps {
                sh '''
                aws ssm send-command \
                  --targets Key=instanceIds,Values=$EC2_INSTANCE_ID \
                  --document-name AWS-RunShellScript \
                  --region $AWS_REGION \
                  --parameters commands="docker stop $CONTAINER_NAME || true && docker rm $CONTAINER_NAME || true && docker pull $DOCKERHUB_USER/$IMAGE_NAME:latest && docker run -d -p 80:80 --name $CONTAINER_NAME $DOCKERHUB_USER/$IMAGE_NAME:latest"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ FULL CI/CD SUCCESS (Build + Push + Deploy)"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
