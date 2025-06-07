pipeline {
    agent any

    environment {
        // Image Docker
        DOCKER_IMAGE = "ghofrane694/msclient"
        // Identifiants
        REGISTRY_CREDENTIALS = credentials('dockerhub-credentials-id')
        GIT_CREDENTIALS = credentials('git-credentials-id')
    }

    stages {
        stage('Clone du repo') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/Ghofrane1233/msclient.git'
            }
        }

        stage('Install dépendances') {
            steps {
                bat 'npm install'
            }
        }

        stage('Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Déploiement sur Minikube') {
            steps {
                script {
                    // Remplacer l’image dans le fichier YAML
                    bat """
                    powershell -Command "(Get-Content k8s-deployment.yaml) -replace 'ghofrane694/msclient:latest', 'ghofrane694/msclient:${BUILD_NUMBER}' | Set-Content k8s-deployment-deploy.yaml"
                    """

                    // Appliquer les ressources Kubernetes
                    bat """
                    kubectl delete deployment msclient --ignore-not-found
                    kubectl delete secret db-secret --ignore-not-found

                    kubectl apply -f db-secret.yaml
                    kubectl apply -f k8s-deployment-deploy.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Déploiement terminé avec succès sur Minikube."
        }
        failure {
            echo "❌ Échec de la pipeline."
        }
    }
}
