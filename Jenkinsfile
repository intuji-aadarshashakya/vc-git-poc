pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')  // Poll every minute (or use webhook)
    }

    environment {
        VERSION_TAG = ''
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                    VERSION_TAG = sh(script: "git describe --tags --exact-match || true", returnStdout: true).trim()
                    if (VERSION_TAG == '') {
                        error "No Tag found on this commit. Skipping pipeline."
                    }
                    echo "Building Version: ${VERSION_TAG}"
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                // Add build steps (npm build, maven, etc.)
            }
        }

        stage('Test') {
            steps {
                echo 'Running Tests...'
                // Add test commands
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying version ${VERSION_TAG}..."
                // Add deployment commands (ssh deploy, docker push, etc.)
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
