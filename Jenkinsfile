pipeline {
    agent any

    environment {
        // Image Docker à construire et publier
        DOCKER_IMAGE = "ghofrane694/msclient"
        
        // Identifiants Jenkins (doivent être créés dans "Manage Credentials")
        REGISTRY_CREDENTIALS = credentials('dockerhub-credentials-id') // Docker Hub
        GIT_CREDENTIALS = 'git-credentials-id' // GitHub Token (type: Username with password)
    }

    stages {
        stage('Clone du dépôt') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/Ghofrane1233/msclient.git'
            }
        }

        stage('Installer les dépendances') {
            steps {
                bat 'npm install'
            }
        }

        stage('Exécuter les tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Construire l’image Docker') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Pousser l’image Docker') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Déploiement sur Minikube') {
            steps {
                script {
                    // ⚠️ Vérifie que PowerShell est installé et que le fichier YAML existe
                    bat """
                    powershell -Command "(Get-Content k8s-deployment.yaml) -replace 'ghofrane694/msclient:latest', 'ghofrane694/msclient:${BUILD_NUMBER}' | Set-Content k8s-deployment-deploy.yaml"
                    """

                    // Déploiement Kubernetes
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
            echo "❌ Échec de la pipeline. Vérifie les logs pour plus d'infos."
        }
    }
}
