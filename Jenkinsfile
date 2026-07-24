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
                    // Yahan hum do tags bana rahe hain: ek 'latest' aur ek build number wala (jaise :3, :4, etc.)
                    app = docker.build("${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}")
                    app.inside {
                        // Image ko 'latest' tag bhi assign kar rahe hain
                    }
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                script {
                    docker.withRegistry("http://${HARBOR_REGISTRY}", "${CREDENTIALS_ID}") {
                        // 1. Build number wali image push hogi (yeh aapka versioned/n-1 backup ban jayegi)
                        app.push("${BUILD_NUMBER}")
                        
                        // 2. Latest tag wali image bhi push hogi
                        app.push("latest")
                    }
                }
            }
        }
    }
}