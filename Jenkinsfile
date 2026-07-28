pipeline {
    agent any
    
    environment {
        HARBOR_REGISTRY = '192.168.5.234'
        PROJECT_NAME    = 'my-portfolio'
        IMAGE_NAME      = 'my-portfolio'
        CREDENTIALS_ID  = 'harbor-creds'
        
        TAG_VERSION     = "${BUILD_NUMBER}"
        TAG_LATEST      = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                script {
                    docker.withRegistry("http://${HARBOR_REGISTRY}", "${CREDENTIALS_ID}") {
                        // 1. Build number wali image push hogi
                        app.push("${BUILD_NUMBER}")
                        
                        // 2. Latest tag wali image push hogi
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Updating Kubernetes deployment with new image tag: ${BUILD_NUMBER}..."
                    
                    # Kubernetes deployment ka image tag update karna (rolling update)
                    kubectl set image deployment/manahil-app manahil-app=${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                    
                    # Rollout status check karna ke deployment successfully complete ho gayi ya nahi
                    kubectl rollout status deployment/manahil-app
                    
                    echo "Kubernetes deployment completed successfully!"
                '''
            }
        }
    }
}