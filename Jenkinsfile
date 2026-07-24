pipeline {
    agent any
    
    environment {
        // Aapke bataye hue IPs ke mutabiq Harbor registry ka address
        HARBOR_REGISTRY = '192.168.5.234'
        PROJECT_NAME    = 'my-portfolio'
        IMAGE_NAME      = 'my-portfolio'
        CREDENTIALS_ID  = 'harbor-login' // Jenkins mein save kiye gaye Harbor credentials ki ID
    }

    stages {
        // Step 1: GitLab (192.168.5.232) se code checkout/download karna
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        // Step 2: Dockerfile ki madad se image build karna
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        // Step 3: Bani hui image ko Harbor (192.168.5.234) par push karna
        stage('Push to Harbor') {
            steps {
                script {
                    docker.withRegistry("http://${HARBOR_REGISTRY}", "${CREDENTIALS_ID}") {
                        app.push()
                    }
                }
            }
        }
    }
}