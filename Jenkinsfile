pipeline {
    agent any

    environment {
        IMAGE_NAME = "zree7/automated_cloud_deployment"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Test') {
            steps {
                echo 'Build succeeded — running smoke check'
                sh '''
                    docker run -d --name lab-web-test -p 8082:80 ${IMAGE_NAME}:latest
                    sleep 3
                    curl -f http://localhost:8082 || (docker logs lab-web-test && exit 1)
                    docker rm -f lab-web-test
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline failed — check console output above.'
        }
        always {
            sh 'docker image prune -f || true'
        }
    }
}
