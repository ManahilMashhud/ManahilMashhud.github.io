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
                    echo "Preparing and updating deployment image tag locally..."
                    
                    # 1. Jenkins apne workspace mein hi deployment.yml ke andar naya image tag daal deta hai
                    REGISTRY_IMAGE="${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}"
                    if [ -f deployment.yml ]; then
                        sed -i "s|image:.*|image: ${REGISTRY_IMAGE}:${TAG_VERSION}|g" deployment.yml
                    fi

                    echo "Copying manifest files to Kubernetes server..."
                    
                    # 2. Remote server par folder ensure karna aur files ko SCP ke zariye bhejna
                    ssh -o StrictHostKeyChecking=no "${APP_USER}@${APP_SERVER}" "mkdir -p /tmp/k8s-manifests"
                    scp -o StrictHostKeyChecking=no pv.yml pvc.yml deployment.yml service.yml "${APP_USER}@${APP_SERVER}:/tmp/k8s-manifests/"

                    echo "Applying manifests on Kubernetes server..."
                    
                    # 3. Remote server par ja kar kubectl apply chala dena
                    ssh -o StrictHostKeyChecking=no "${APP_USER}@${APP_SERVER}" \
                        "NAMESPACE='${K8S_NAMESPACE}' \
                         bash -s" <<'REMOTE_SCRIPT'

                        set -e
                        cd /tmp/k8s-manifests

                        kubectl apply -f pv.yml --namespace "$NAMESPACE" || true
                        kubectl apply -f pvc.yml --namespace "$NAMESPACE" || true
                        kubectl apply -f deployment.yml --namespace "$NAMESPACE"
                        kubectl apply -f service.yml --namespace "$NAMESPACE"

                        kubectl rollout status deployment/my-portfolio-deployment --namespace "$NAMESPACE" --timeout=180s

                        echo "Kubernetes deployment completed successfully!"
REMOTE_SCRIPT
                '''
            }
        }
    }
}