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
                        app.push("${BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Connecting to Kubernetes server to update deployment image tag: ${BUILD_NUMBER}..."
                    
                    # SSH ke zariye 235 server par jaakar kubectl command chalayenge
                    ssh -o StrictHostKeyChecking=no devops@192.168.5.235 "kubectl set image deployment/manahil-app manahil-app=${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} && kubectl rollout status deployment/manahil-app"
                    
                    echo "Kubernetes deployment completed successfully!"
                '''
            }
        }
    }
}