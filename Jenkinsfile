pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ghofrane694/msclient"
        REGISTRY_CREDENTIALS_ID = 'docker-credentials-id'
        GIT_CREDENTIALS_ID = 'git-credentials-id'
    }

    stages {
        stage('Cloner le dépôt Git') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/Ghofrane1233/msclient.git', branch: 'main'
            }
        }

        stage('Installation des dépendances') {
            steps {
                bat 'npm install' // Remplacer par sh si sur un agent Linux
            }
        }

        stage('Tests') {
            steps {
                bat 'npm test' // Remplacer par sh si sur un agent Linux
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.BUILT_IMAGE_ID = docker.build(env.DOCKER_IMAGE).id
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', env.REGISTRY_CREDENTIALS_ID) {
                        docker.image(env.DOCKER_IMAGE).push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://192.168.217.133:8443']) {
                            bat 'kubectl apply -f K8s --validate=false'
                        }
                    } catch (Exception e) {
                        error "Kubernetes deployment failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Déploiement terminé avec succès sur Minikube."
        }
        failure {
            echo "❌ La pipeline a échoué. Vérifiez les logs pour plus de détails."
        }
    }
}
