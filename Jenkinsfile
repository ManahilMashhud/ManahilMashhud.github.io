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
                    echo "Deploying latest image to server using Docker Compose..."
                    
                    # 1. Purana container roko aur hatao
                    docker stop portfolio-container || true
                    docker rm portfolio-container || true
                    
                    # 2. Agar port 8081 par koi aur stray container chalu ho toh usko free karo
                    docker ps -q --filter "publish=8081" | xargs -r docker stop || true
                    docker ps -a -q --filter "publish=8081" | xargs -r docker rm || true
                    
                    # 3. Purani local latest image hatao taake cache ka masla na ho
                    docker rmi ${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:latest || true
                    
                    # 4. Docker Compose ke zariye naye version ki image pull karke container live karo (hyphen fix)
                    docker-compose pull
                    docker-compose up -d --force-recreate
                    
                    echo "Deployment completed successfully!"
                '''
            }
        }
    }
}