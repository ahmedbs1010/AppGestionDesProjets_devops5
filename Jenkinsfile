pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'rbns10'      // à remplacer
        TAG            = "${BUILD_NUMBER}"
        BACKEND_IMAGE  = "${DOCKERHUB_USER}/gestion-projets-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/gestion-projets-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Création des images') {
            steps {
                sh 'docker compose build'
                sh 'docker images | grep gestion-projets'
            }
        }

        stage('Création des conteneurs') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d'
                sh '''
                  for i in $(seq 1 30); do
                    curl -sf http://localhost:8089/entreprise/all && { echo " -> Stack OK"; exit 0; }
                    echo "En attente du backend... ($i)"; sleep 5
                  done
                  docker compose logs backend; exit 1
                '''
                sh 'docker compose ps'
            }
        }

        stage('Credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                }
            }
        }

        stage('Push image') {
            steps {
                sh '''
                  docker tag $BACKEND_IMAGE:$TAG  $BACKEND_IMAGE:latest
                  docker tag $FRONTEND_IMAGE:$TAG $FRONTEND_IMAGE:latest
                  docker push $BACKEND_IMAGE:$TAG
                  docker push $BACKEND_IMAGE:latest
                  docker push $FRONTEND_IMAGE:$TAG
                  docker push $FRONTEND_IMAGE:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
