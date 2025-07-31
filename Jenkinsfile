pipeline {
    agent any

    triggers {
        githubPush()  // Trigger on push events
    }

    environment {
        DOCKER_REGISTRY = 'registry.digitalocean.com/intujii'
        IMAGE_NAME = 'mkdocs'
    }

    stages {
        stage('Check if Tag Push') {
            steps {
                script {
                    env.GIT_TAG = sh(script: "git describe --tags --exact-match 2>/dev/null || echo ''", returnStdout: true).trim()
                    if (!env.GIT_TAG) {
                        echo "This push is not a tag. Skipping build."
                        currentBuild.result = 'NOT_BUILT'
                        error("No tag found on this commit. Exiting pipeline.")
                    } else {
                        echo "Building for Tag: ${env.GIT_TAG}"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'CONTAINER-REGESTRY-TOKEN', variable: 'DO_TOKEN')]) {
                        sh '''
                        doctl auth init --access-token "$DO_TOKEN"
                        doctl registry login

                        echo "Building Docker Image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${GIT_TAG}"
                        docker build --rm --squash --no-cache -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${GIT_TAG} -f Dockerfile .
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${GIT_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    echo "Deploying Version: ${GIT_TAG} to Kubernetes"
                    kubectl config use-context do-syd1-intuji

                    export VERSION_TAG=${GIT_TAG}
                    envsubst < $WORKSPACE/k8s/deploy.yaml | kubectl apply -f - --namespace=default
                    '''
                }
            }
        }

        stage('Release Notification') {
            steps {
                echo "Version ${GIT_TAG} deployed successfully!"
                // Optional: Slack/Webhook Notification
            }
        }
    }
}
