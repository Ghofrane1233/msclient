pipeline {
    agent any

    environment {
        // Nom de l'image Docker
        DOCKER_IMAGE = "ghofrane694/msclient"
        
        // Identifiants
        REGISTRY_CREDENTIALS = credentials('dockerhub-credentials-id')
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
                bat 'npm install'
            }
        }

        stage('Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Construction de l’image Docker') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push de l’image Docker sur Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${REGISTRY_CREDENTIALS}") {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Déploiement sur Minikube') {
            steps {
                script {
                    // Mise à jour de l’image dans le fichier YAML
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
            echo "❌ La pipeline a échoué. Vérifiez les logs pour plus de détails."
        }
    }
}
