pipeline {
    agent any

    environment {
        // Docker image info
        DOCKER_IMAGE = "nambi533/react-nginx-docker"
        DOCKER_TAG   = "latest"
        DOCKER_CREDS = "dockerhub-creds"

        // EC2 info
        EC2_USER = "ec2-user"
        EC2_IP   = "3.110.131.17"
        SSH_KEY  = "ec2-key"

        APP_NAME = "react-ui"
        HOST_PORT = "80"
        CONTAINER_PORT = "80"

        // Kubernetes info
        KUBE_CONFIG_CRED = "kubeconfig-prod" // Jenkins credential with kubeconfig file
        K8S_MANIFEST_DIR = "k8s"            // Kubernetes manifests folder in repo
    }

    stages {

        stage('Checkout Source') {
            steps {
                git url: 'https://github.com/sasikumar2022k/react-nginx-docker.git', branch: 'master'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        echo \$DOCKER_TOKEN | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([SSH_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                            docker run -d --name ${APP_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_CRED}", variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f ${K8S_MANIFEST_DIR}/deployment.yaml
                        kubectl apply -f ${K8S_MANIFEST_DIR}/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ React UI deployed successfully on EC2 and Kubernetes!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}

