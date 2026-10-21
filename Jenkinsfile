pipeline {
    agent any

    environment {
        IMAGE_NAME = 'shravanipatwardhan/student-management-system'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shravani172206-ctrl/student-management-system-cicd.git'
            }
        }

        stage('Validate Application') {
            steps {
                sh 'python3 -m py_compile backend/app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -e

                    kubectl set image deployment/student-management \
                      student-management=${IMAGE_NAME}:${BUILD_NUMBER}

                    kubectl rollout status deployment/student-management \
                      --timeout=120s

                    kubectl get deployment student-management
                '''
            }
        }
    }

    post {
        always {
            echo 'CI/CD pipeline execution finished.'
        }

        success {
            echo 'Build, image publishing, and Kubernetes deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs for details.'
        }
    }
}
