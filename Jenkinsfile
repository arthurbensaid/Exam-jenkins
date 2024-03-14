pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "arthurbensaid"
DOCKER_IMAGE_NGINX = "exam-jenkins"
DOCKER_IMAGE_CAST = "cast"
DOCKER_IMAGE_CAST_DB = "cast-db"
DOCKER_IMAGE_MOVIE = "movie"
DOCKER_IMAGE_MOVIE_DB = "movie-db"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f $DOCKER_IMAGE_CAST
                 docker rm -f $DOCKER_IMAGE_CAST_DB
                 docker rm -f $DOCKER_IMAGE_MOVIE
                 docker rm -f $DOCKER_IMAGE_MOVIE_DB
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./microservices/cast
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_DB:$DOCKER_TAG ./microservices/cast-db
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./microservices/movie
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_DB:$DOCKER_TAG ./microservices/movie-db
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8002:8000 --name $DOCKER_IMAGE_CAST $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker run -d -p 81:5432 --name $DOCKER_IMAGE_CAST_DB $DOCKER_ID/$DOCKER_IMAGE_CAST_DB:$DOCKER_TAG
                    docker run -d -p 8001:8000 --name $DOCKER_IMAGE_MOVIE $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    docker run -d -p 83:5432 --name $DOCKER_IMAGE_MOVIE_DB $DOCKER_ID/$DOCKER_IMAGE_MOVIE_DB:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST_DB:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE_DB:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp microservices/movie/movie/values.yaml values_movie.yml
                cat values_movie.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie.yml
                helm upgrade --install movie-chart ./microservices/movie --values=values_movie.yml --namespace dev
                cp microservices/movie-db/movie-db/values.yaml values_movie-db.yml
                cat values_movie-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie-db.yml
                helm upgrade --install movie-db-chart ./microservices/movie-db --values=values_movie-db.yml --namespace dev
                cp microservices/cast/cast/values.yaml values_cast.yml
                cat values_cast.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast.yml
                helm upgrade --install cast-chart ./microservices/cast --values=values_cast.yml --namespace dev
                cp microservices/cast-db/cast-db/values.yaml values_cast-db.yml
                cat values_cast-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast-db.yml
                helm upgrade --install cast-db-chart ./microservices/cast-db --values=values_cast-db.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp microservices/movie/movie/values.yaml values_movie.yml
                cat values_movie.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie.yml
                helm upgrade --install movie-chart ./microservices/movie --values=values_movie.yml --namespace QA
                cp microservices/movie-db/movie-db/values.yaml values_movie-db.yml
                cat values_movie-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie-db.yml
                helm upgrade --install movie-db-chart ./microservices/movie-db --values=values_movie-db.yml --namespace QA
                cp microservices/cast/cast/values.yaml values_cast.yml
                cat values_cast.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast.yml
                helm upgrade --install cast-chart ./microservices/cast --values=values_cast.yml --namespace QA
                cp microservices/cast-db/cast-db/values.yaml values_cast-db.yml
                cat values_cast-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast-db.yml
                helm upgrade --install cast-db-chart ./microservices/cast-db --values=values_cast-db.yml --namespace QA
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp microservices/movie/movie/values.yaml values_movie.yml
                cat values_movie.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie.yml
                helm upgrade --install movie-chart ./microservices/movie --values=values_movie.yml --namespace staging
                cp microservices/movie-db/movie-db/values.yaml values_movie-db.yml
                cat values_movie-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie-db.yml
                helm upgrade --install movie-db-chart ./microservices/movie-db --values=values_movie-db.yml --namespace staging
                cp microservices/cast/cast/values.yaml values_cast.yml
                cat values_cast.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast.yml
                helm upgrade --install cast-chart ./microservices/cast --values=values_cast.yml --namespace staging
                cp microservices/cast-db/cast-db/values.yaml values_cast-db.yml
                cat values_cast-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast-db.yml
                helm upgrade --install cast-db-chart ./microservices/cast-db --values=values_cast-db.yml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp microservices/movie/movie/values.yaml values_movie.yml
                cat values_movie.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie.yml
                helm upgrade --install movie-chart ./microservices/movie --values=values_movie.yml --namespace prod
                cp microservices/movie-db/movie-db/values.yaml values_movie-db.yml
                cat values_movie-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_movie-db.yml
                helm upgrade --install movie-db-chart ./microservices/movie-db --values=values_movie-db.yml --namespace prod
                cp microservices/cast/cast/values.yaml values_cast.yml
                cat values_cast.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast.yml
                helm upgrade --install cast-chart ./microservices/cast --values=values_cast.yml --namespace prod
                cp microservices/cast-db/cast-db/values.yaml values_cast-db.yml
                cat values_cast-db.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_cast-db.yml
                helm upgrade --install cast-db-chart ./microservices/cast-db --values=values_cast-db.yml --namespace prod
                '''
                }
            }

        }

}
}
