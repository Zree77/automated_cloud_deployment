pipeline {
    agent any

    environment {
        IMAGE_NAME = "zree7/automated_cloud_deployment"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"

                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Test') {
            steps {
                echo 'Running Docker container smoke test'

                sh '''
                    docker rm -f lab-web-test 2>/dev/null || true

                    docker run -d \
                        --name lab-web-test \
                        -p 8082:80 \
                        ${IMAGE_NAME}:latest

                    sleep 5

                    curl -f http://localhost:8082

                    docker rm -f lab-web-test
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo 'Connecting to Ansible server and deploying application'

                sh '''
                    ssh \
                        -i /var/lib/jenkins/.ssh/ansible_key \
                        -o StrictHostKeyChecking=no \
                        ubuntu@15.207.98.175 \
                        "ansible-playbook \
                        -i /home/ubuntu/inventory.ini \
                        /home/ubuntu/playbook.yaml"
                '''
            }
        }
    }

    post {
        success {
            echo '============================================'
            echo 'PIPELINE SUCCESSFUL'
            echo '============================================'
            echo 'Docker image built successfully'
            echo 'Docker image pushed to Docker Hub'
            echo 'Ansible deployment completed successfully'
            echo 'Website: http://65.1.112.195:8081'
            echo '============================================'
        }

        failure {
            echo '============================================'
            echo 'PIPELINE FAILED'
            echo 'Check the console output above.'
            echo '============================================'
        }

        always {
            sh 'docker rm -f lab-web-test 2>/dev/null || true'
            sh 'docker image prune -f || true'
        }
    }
}
