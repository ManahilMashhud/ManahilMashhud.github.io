pipeline {
    agent any
    
    environment {
        HARBOR_REGISTRY = '192.168.5.234'
        PROJECT_NAME    = 'my-portfolio'
        IMAGE_NAME      = 'my-portfolio'
        CREDENTIALS_ID  = 'harbor-creds'
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
                        // Build number aur latest dono push honge
                        app.push("${BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                    echo "Updating and restarting container using Docker Compose..."
                    
                    # Purani local latest image hatao taake cache ka masla na ho
                    docker rmi 192.168.5.234/my-portfolio/my-portfolio:latest || true
                    
                    # Docker compose ke zariye nayi image pull karke container recreate karo
                    docker compose pull
                    docker compose up -d --force-recreate
                    
                    echo "Deployment completed successfully!"
                '''
            }
        }
    }
}