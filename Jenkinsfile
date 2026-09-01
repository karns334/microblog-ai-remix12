pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = "microblog-ai"
        CONTAINER_NAME = "microblog-ai"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Getting latest code from main..."
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    echo "Checking repository..."
                    test -f Dockerfile
                    docker --version
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    echo "Building Docker image..."

                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                    .
                '''
            }
        }

        stage('Test Image') {
            steps {
                sh '''
                    echo "Testing image..."

                    docker image inspect \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying application..."

                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    --restart unless-stopped \
                    -p 80:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5

                    docker ps

                    curl -f http://localhost
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Application deployed"
        }

        failure {
            echo "FAILED: Pipeline failed"
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
