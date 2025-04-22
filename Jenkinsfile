pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://127.0.0.1:8200'  // Or your Vault IP
        DOCKER_IMAGE = "yuvraj4/node-app:latest"
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
                git 'https://github.com/yuvrajm-14/ci-cd-nodejs-mysql.git'
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

