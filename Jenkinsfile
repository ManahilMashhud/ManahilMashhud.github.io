pipeline {
    agent any
    
    environment {
        HARBOR_REGISTRY = '192.168.5.232:8085'
        PROJECT_NAME    = 'my-portfolio'
        IMAGE_NAME      = 'my-portfolio'
        CREDENTIALS_ID  = 'harbor-login' // Yeh aapke Jenkins ke credentials ki ID hai
    }

    stages {
        // Step 1: GitLab se code apne paas download karna
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        // Step 2: Dockerfile ki madad se image build karna
        stage('Build Docker Image') {
            steps {
                script {
                    // Yahan image ka tag build number ke sath banega (misal: my-portfolio:1)
                    app = docker.build("${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        // Step 3: Bani hui image ko Harbor registry par push karna
        stage('Push to Harbor') {
            steps {
                script {
                    // Jenkins mein save kiye gaye credentials use karke login aur push karega
                    docker.withRegistry("http://${HARBOR_REGISTRY}", "${CREDENTIALS_ID}") {
                        app.push()
                    }
                }
            }
        }
    }
}