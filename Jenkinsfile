pipeline {
    environment {
        // Déclaration des variables d'environnement
        DOCKER_ID = "parakltos"
        MOVIE_SERVICE_IMAGE = "movie-service"
        CAST_SERVICE_IMAGE = "cast-service"
        NGINX_IMAGE = "nginx:latest"
        POSTGRES_IMAGE = "postgres:12.1-alpine"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any 
    stages {
        stage('Build Movie Service') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG ./movie-service
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Build Cast Service') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG ./cast-service
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes Dev') {
            steps {
                script {
                    deployToKubernetes("dev")
                }
            }
        }
        stage('Deploy to Kubernetes Staging') {
            steps {
                script {
                    deployToKubernetes("staging")
                }
            }
        }
        stage('Production Deployment Approval') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
            }
        }
        stage('Deploy to Kubernetes Prod') {
            steps {
                script {
                    deployToKubernetes("prod")
                }
            }
        }
    }
}

// Définir une méthode de déploiement réutilisable
def deployToKubernetes(String environment) {
    KUBECONFIG = credentials("config")
    sh '''
    rm -Rf .kube
    mkdir .kube
    echo
    cp fastapi/values.yaml values.yml
 
    sed -i "s+image: movie_service+image: $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG+g" values.yml
    sed -i "s+image: cast_service+image: $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG+g" values.yml
    sed -i "s+image: nginx+image: $NGINX_IMAGE+g" values.yml
    sed -i "s+image: postgres+image: $POSTGRES_IMAGE+g" values.yml
 
   helm upgrade --install app fastapi --values=values.yml --namespace prod
    '''
}