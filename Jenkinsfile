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

        stage('Deploy to Server') {
            steps {
                sh '''
                    echo "Deploying latest image to server on port 8081..."
                    
                    # 1. Purana container roko aur hatao
                     docker stop portfolio-container || true
                     docker rm portfolio-container || true
                    
                    # 2. Purani local latest image hatao taake cache ka masla na ho
                     docker rmi 192.168.5.234/my-portfolio/my-portfolio:latest || true
                    
                    # 3. Harbor se bilkul nayi latest image pull karo
                     docker pull ${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:latest
                    
                    # 4. Port 8081 par naya container live karo
                     docker run -d --name portfolio-container -p 8081:80 --restart always ${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:latest
                    
                    echo "Deployment completed successfully!"
                '''
            }
        }
    }
}