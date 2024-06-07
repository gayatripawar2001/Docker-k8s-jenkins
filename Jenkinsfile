pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id') // Ensure this matches the ID in Jenkins
        KUBE_CONFIG = credentials('minikube-kubeconfig') // Ensure this matches the kubeconfig credentials ID in Jenkins
    }
    stages {
        stage('Build') {
            steps {
                script {
                    // Set the path to your Dockerfile and application
                    def appDir = 'D:\\SRE-DevOps\\POCs\\Flask-app' // Update this to your actual directory
                    dir(appDir) {
                        dockerImage = docker.build("guser1/flask-app:${env.BUILD_ID}") // Replace with your DockerHub username and repository if needed
                    }
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials-id') {
                        // Set Docker client timeout
                        withEnv(["DOCKER_CLIENT_TIMEOUT=300", "COMPOSE_HTTP_TIMEOUT=300"]) {
                            retry(3) {
                                dockerImage.push("latest")
                                dockerImage.push("${env.BUILD_ID}")
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'minikube-kubeconfig', variable: 'KUBECONFIG')]) {
                        // Set the path to your Kubernetes manifests
                        def k8sManifestsDir = 'D:\\SRE-DevOps\\POCs\\Flask-app\\k8s-manifests' // Update this to your actual directory
                        retry(3) {
                            bat """
                            kubectl apply -f ${k8sManifestsDir}\\deployment.yaml --kubeconfig=%KUBECONFIG% || exit 1
                            kubectl apply -f ${k8sManifestsDir}\\service.yaml --kubeconfig=%KUBECONFIG% || exit 1
                            """
                        }
                    }
                }
            }
        }
    }
}
