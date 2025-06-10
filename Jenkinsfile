pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ghofrane694/msclient"
        REGISTRY_CREDENTIALS_ID = 'docker-hub-credentials-id'
    
    }

    stages {
        stage('Cloner le dépôt Git') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/Ghofrane1233/msclient.git', branch: 'main'
            }
        }

        stage('Installation des dépendances') {
            steps {
                bat 'npm install' 
            }
        }

        stage('Tests') {
            steps {
                bat 'npm test' 
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
                        withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://127.0.0.1:51662']) {
                            bat 'kubectl apply -f db-secret.yaml --validate=false'
                            bat 'kubectl apply -f k8s/deployment.yaml --validate=false'
                            bat 'kubectl apply -f k8s/service.yaml --validate=false'
                        }
                    } catch (Exception e) {
                        error "Kubernetes deployment failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage(' Monitoring ') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://127.0.0.1:54825']) {
                        // Déployer Prometheus
                        bat 'kubectl apply -f monitoring/prometheus-config.yaml'
                        bat 'kubectl apply -f monitoring/prometheus-deployment.yaml'
                        bat 'kubectl apply -f monitoring/prometheus-service.yaml'

                        // Déployer Grafana
                        bat 'kubectl apply -f monitoring/grafana-deployment.yaml'
                        bat 'kubectl apply -f monitoring/grafana-service.yaml'
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
