pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')  // Poll every minute (or use webhook)
    }

    environment {
        DOCKER_REGISTRY = 'registry.digitalocean.com/intuji'
        IMAGE_NAME = 'vc-git-poc'
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
                    VERSION_TAG = sh(script: "git describe --tags --exact-match || echo 'latest'", returnStdout: true).trim()
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
                        docker build --rm --squash --no-cache -t registry.digitalocean.com/intuji/vc-git-poc:${VERSION_TAG} -f Dockerfile .
                        docker push registry.digitalocean.com/intuji/vc-git-poc:${VERSION_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
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
