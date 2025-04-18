pipeline {
    agent any

    environment {
        // Define environment variables
        DOCKER_IMAGE = 'atomic-crm'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'your-registry' // Replace with your registry
        KUBERNETES_NAMESPACE = 'supabase'
        KUBECONFIG = credentials('kubeconfig') // Jenkins credential ID for kubeconfig
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from repository
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Node.js dependencies
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login to Docker registry and push image
                    docker.withRegistry('https://${DOCKER_REGISTRY}', 'docker-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        // Also push as latest
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    // Update the deployment with new image
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        // Update image in deployment
                        sh """
                            kubectl set image deployment/atomic-crm \
                            atomic-crm=${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} \
                            -n ${KUBERNETES_NAMESPACE}
                        """

                        // Verify rollout status
                        sh """
                            kubectl rollout status deployment/atomic-crm \
                            -n ${KUBERNETES_NAMESPACE}
                        """
                    }
                }
            }
        }

        stage('Verify Supabase Services') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        // Check if all required services are running
                        sh """
                            kubectl get pods -n ${KUBERNETES_NAMESPACE}
                            kubectl get services -n ${KUBERNETES_NAMESPACE}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            // Notify on successful deployment
            echo 'Deployment successful!'
            // Add notification steps (e.g., Slack, email) here
        }
        failure {
            // Notify on deployment failure
            echo 'Deployment failed!'
            // Add notification steps (e.g., Slack, email) here
        }
        always {
            // Clean workspace
            cleanWs()
        }
    }
} 