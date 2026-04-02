pipeline {
    agent any

    environment {
        IMAGE_NAME = "portfolio-image"
        CONTAINER_NAME = "portfolio-container"
        EC2_INSTANCE_ID = "i-02ac3288e6eacc6f9" // replace with your EC2 ID
        AWS_REGION = "ap-south-1"               // replace with your region
    }

    stages {
        stage('Deploy to EC2 via SSM') {
            steps {
                sh """
                aws ssm send-command \
                  --targets Key=instanceIds,Values=${EC2_INSTANCE_ID} \
                  --document-name AWS-RunShellScript \
                  --region ${AWS_REGION} \
                  --comment "Deploy portfolio container" \
                  --parameters 'commands=[
                    "docker stop ${CONTAINER_NAME} || true",
                    "docker rm ${CONTAINER_NAME} || true",
                    "docker pull ${IMAGE_NAME} || true",
                    "docker run -d -p 80:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}"
                  ]'
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}