pipeline {
    agent {
        label 'Node1'
    }

    environment {
        APP_NAME = 'complete-production-e2e-pipeline'
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag from CI')
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM (SSH)') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:Sudhakar20000/e-to-e-deploy.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Workspace:"
                    pwd

                    echo "Files:"
                    ls -la

                    echo "Deployment file:"
                    cat deployment.yaml
                '''
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                sh '''
                    echo "Updating image tag to: ${IMAGE_TAG}"

                    sed -i "s#sudhakar20000/complete-e2e-pipeline:.*#sudhakar20000/complete-e2e-pipeline:${IMAGE_TAG}#g" deployment.yaml

                    echo "Updated deployment.yaml:"
                    cat deployment.yaml
                '''
            }
        }

        stage('Commit & Push Changes (SSH)') {
            steps {
                sh '''
                    git config --global user.name "Sudhakar20000"
                    git config --global user.email "Sudhakar@man.cloud"

                    git add deployment.yaml

                    git commit -m "Updated Deployment Manifest with ${IMAGE_TAG}" || echo "No changes to commit"

                    git push origin main
                '''
            }
        }
    }
}
