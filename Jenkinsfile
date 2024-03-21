pipeline {
    environment { 
        DOCKER_ID = "parakltos" 
        MOVIE_SERVICE_IMAGE = "movie-service"
        CAST_SERVICE_IMAGE = "cast-service"
        NGINX_IMAGE = "nginx"
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
                    '''
                }
            }
        }
        stage('Build Cast Service') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Docker Push Movie Service') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Docker Push Cast Service') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deployment to Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployToKubernetes("dev")
                }
            }
        }
        stage('Deployment to Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
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
        stage('Deployment to Prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
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
    sh '''
    rm -Rf .kube
    mkdir .kube
    cat $KUBECONFIG > .kube/config
    cp fastapi/values.yaml values.yml
    sed -i "s+tag: movie-service.*+tag: movie-service:$DOCKER_TAG+g" values.yml
    sed -i "s+tag: cast-service.*+tag: cast-service:$DOCKER_TAG+g" values.yml
    helm upgrade --install app fastapi --values=values.yml --namespace prod
    '''
}
