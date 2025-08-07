pipeline {
    agent any

    environment {
        IMAGE_NAME = "ci-cd-node-image"
        DOCKERHUB_USERNAME = "aravind310730"
        RELEASE_NAME = "my-app"
        CHART_PATH = "./helm-chart"
        // Add KUBECONFIG if needed
        // KUBECONFIG = '/path/to/kubeconfig'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/aravind30122003/project-1-cicd.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest || true"
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $DOCKER_USERNAME/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        // Add the kubectl check stage here, same indentation as others
        stage('Kubectl Check') {
            steps {
                sh 'kubectl cluster-info'
                sh 'kubectl get nodes'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                helm upgrade --install $RELEASE_NAME $CHART_PATH --set image.tag=latest
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}

