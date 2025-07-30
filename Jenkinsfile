pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'registry.digitalocean.com/intuji'
        IMAGE_NAME = 'mkdocs'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

    stage('Get Version Tag') {
            steps {
                script {
                    // Get the Git Tag that triggered this build
                    VERSION_TAG = sh(script: "git describe --tags --exact-match 2>/dev/null || echo ''", returnStdout: true).trim()
                    if (!VERSION_TAG) {
                        error "No Git tag found on this commit. Skipping pipeline."
                    }
                    echo "VERSION_TAG = ${VERSION_TAG}"
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                script {
                    withCredentials([string(credentialsId: 'CONTAINER-REGESTRY-TOKEN', variable: 'DO_TOKEN')]) {
                        sh '''
                        doctl auth init --access-token $DO_TOKEN
                        doctl registry login

                        VERSION_TAG=$(git describe --tags --exact-match 2>/dev/null)
                        echo "VERSION_TAG = ${VERSION_TAG}"
                        
                        docker build --rm --squash --no-cache -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION_TAG} -f Dockerfile .
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                    export VERSION_TAG=$(git describe --tags --exact-match)
                    kubectl config use-context do-syd1-intuji
                    envsubst < $WORKSPACE/k8s/deploy.yaml | kubectl apply -f - --namespace=default
                    '''
                }
            }
        }

        stage('Tag Release Notification') {
            steps {
                echo "Version ${VERSION_TAG} deployed successfully!"
                // Optional: Slack/Webhook notifications
            }
        }
    }
}
