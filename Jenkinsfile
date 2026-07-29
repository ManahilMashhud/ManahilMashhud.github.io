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
        K8S_DEPLOYMENT  = 'manahil'
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
                    echo "Connecting to Kubernetes server to update deployment..."
                    
                    ssh -o StrictHostKeyChecking=no "${APP_USER}@${APP_SERVER}" \
                        "NAMESPACE='${K8S_NAMESPACE}' \
                         NEW_IMAGE='${HARBOR_REGISTRY}/${PROJECT_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}' \
                         DEPLOYMENT='${K8S_DEPLOYMENT}' \
                         bash -s" <<'REMOTE_SCRIPT'

                        set -e
                        
                        # Container ka naam dynamically nikalna taake koi error na aaye
                        CONTAINER_NAME=$(kubectl get deployment "$DEPLOYMENT" \
                            --namespace "$NAMESPACE" \
                            -o jsonpath='{.spec.template.spec.containers[0].name}')

                        if [ -z "$CONTAINER_NAME" ]; then
                            echo "Could not determine the container name."
                            exit 1
                        fi

                        echo "Updating deployment with container: $CONTAINER_NAME and image: $NEW_IMAGE"

                        kubectl set image \
                            deployment/"$DEPLOYMENT" \
                            "$CONTAINER_NAME=$NEW_IMAGE" \
                            --namespace "$NAMESPACE"

                        kubectl rollout status \
                            deployment/"$DEPLOYMENT" \
                            --namespace "$NAMESPACE" \
                            --timeout=180s

                        echo "Kubernetes deployment completed successfully!"
REMOTE_SCRIPT
                '''
            }
        }
    }
}