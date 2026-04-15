pipeline {
    agent {
        label 'Node1'
    }

    environment {
        APP_NAME = 'complete-production-e2e-pipeline'
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/Sudhakar20000/e-to-e-deploy'
            }
        }

        stage('Update the Deployment Tags') {
            steps {
                sh '''
                    cat deployment.yaml
                    sed -i "s/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g" deployment.yaml
                    cat deployment.yaml
                '''
            }
        }

        stage('Push the changed deployment file to Git') {
            steps {
                sh '''
                    git config --global user.name "Sudhakar20000"
                    git config --global user.email "Sudhakar@man.cloud"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                '''

                withCredentials([
                    gitUsernamePassword(
                        credentialsId: 'github',
                        gitToolName: 'Default'
                    )
                ]) {
                    sh 'git push https://github.com/Sudhakar20000/e-to-e-deploy main'
                }
            }
        }
    }
}
