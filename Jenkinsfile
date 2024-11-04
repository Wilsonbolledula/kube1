pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub'                  // Update with your Docker credentials ID
        DOCKER_IMAGE = 'wilsonbolledula/my-kube1'            // Your Docker Hub repository and image tag
        HTTP_PROXY = 'http://your.proxy.server:port'         // Update with your actual proxy, if any
        HTTPS_PROXY = 'http://your.proxy.server:port'        // Add HTTPS proxy if needed
        NO_PROXY = 'localhost,127.0.0.1'                      // Add any necessary non-proxy hosts
    }

    stages {
        stage('Build') {
            steps {
                script {
                    bat "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                        bat "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Start Minikube') {
            steps {
                script {
                    // Stop and delete any previous Minikube instance
                    bat '''
                    minikube stop || true
                    minikube delete || true
                    '''

                    // Configure Minikube with proxy settings and alternative image repository
                    def minikubeStartCommand = """
                    minikube start --image-repository=mirror.gcr.io \
                        --docker-env HTTP_PROXY=%HTTP_PROXY% \
                        --docker-env HTTPS_PROXY=%HTTPS_PROXY% \
                        --docker-env NO_PROXY=%NO_PROXY% \
                        --extra-config=apiserver.enable-admission-plugins="" \
                        --addons=none
                    """

                    // Retry Minikube start if it fails initially
                    retry(3) {
                        bat minikubeStartCommand
                    }
                }
            }
        }

        stage('Enable Minikube Addons') {
            steps {
                script {
                    // Enable required Minikube addons individually with validation disabled
                    bat '''
                    minikube addons enable storage-provisioner --validate=false || true
                    minikube addons enable default-storageclass --validate=false || true
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply the Kubernetes YAML file from your repository
                    bat '''
                    minikube kubectl -- apply -f https://raw.githubusercontent.com/Wilsonbolledula/kube1/main/your-kubernetes-file.yaml
                    '''
                }
            }
        }
    }
}
