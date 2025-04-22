pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://localhost:8200'  // Or your Vault IP
        DOCKER_IMAGE = "your-dockerhub-username/node-app:latest"
    }

    stages {
        stage('Fetch DockerHub Secrets from Vault') {
            steps {
                script {
                    def secrets = vaultSecrets([
                        [path: 'secret/dockerhub', engineVersion: 2, secretValues: [
                            [envVar: 'DOCKER_USERNAME', vaultKey: 'username'],
                            [envVar: 'DOCKER_PASSWORD', vaultKey: 'password']
                        ]]
                    ])
                }
            }
        }

        stage('Checkout') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                docker push ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }

        stage('ArgoCD Sync') {
            steps {
                sh 'argocd app sync node-app'  // optional: use webhook for auto-sync
            }
        }
    }
}

