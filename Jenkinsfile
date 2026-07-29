pipeline {
    agent any
    
    environment {
        HARBOR_REGISTRY = '192.168.5.234'
        PROJECT_NAME    = 'my-portfolio'
        IMAGE_NAME      = 'my-portfolio'
        CREDENTIALS_ID  = 'harbor-creds'
        
        TAG_VERSION     = "${BUILD_NUMBER}"
        TAG_LATEST      = "latest"

        APP_SERVER      = '192.168.5.235'
        APP_USER        = 'devops'
        K8S_NAMESPACE   = 'death'
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
                    set -e
                    echo "Connecting to Kubernetes server to apply manifests..."
                    
                    ssh -o StrictHostKeyChecking=no "${APP_USER}@${APP_SERVER}" \
                        "NAMESPACE='${K8S_NAMESPACE}' \
                         NEW_TAG='${TAG_VERSION}' \
                         REGISTRY='${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}' \
                         bash -s" <<'REMOTE_SCRIPT'

                        set -e
                        
                        # Agar workspace par files hain, toh deployment file ke andar naya tag update kar dete hain
                        if [ -f deployment.yml ]; then
                            sed -i "s|image:.*|image: $REGISTRY:$NEW_TAG|g" deployment.yml
                        fi

                        # Kubernetes manifest files apply karna
                        kubectl apply -f pv.yml --namespace "$NAMESPACE" || true
                        kubectl apply -f pvc.yml --namespace "$NAMESPACE" || true
                        kubectl apply -f deployment.yml --namespace "$NAMESPACE"
                        kubectl apply -f service.yml --namespace "$NAMESPACE"

                        # Rollout status check karna taake confirm ho jaye ke app chal gayi hai
                        kubectl rollout status deployment/my-portfolio-deployment --namespace "$NAMESPACE" --timeout=180s

                        echo "Kubernetes deployment completed successfully!"
REMOTE_SCRIPT
                '''
            }
        }
    }
}