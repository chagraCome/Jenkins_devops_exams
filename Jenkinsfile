pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "asmachagra" // replace this with your docker-id
DOCKER_IMAGE_CAST = "cast_servic"
DOCKER_IMAGE_MOVIE = "movie-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                
                docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                sleep 6
                '''
                }
                 script {
                sh '''
               
                docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                sleep 6
                '''
                }
            }
        }
        stage(' Docker run'){ // run container from our built image
                steps {
                    script {
                    sh '''
                    docker rm --force cast-db
                    docker rm --force cast-service 
                    docker rm --force movie_db
                    docker rm --force movie-service
                    docker rm --force nginx
                    docker run -d --rm --name cast-db  --env POSTGRES_USER=cast_db_username --env POSTGRES_PASSWORD=cast_db_password --env POSTGRES_DB=cast_db_dev postgres:12.1-alpine 
                    docker run -d -p 8002:8000 --name cast-service --rm --env DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG 

                    docker run -d --name movie_db --rm --env POSTGRES_USER=movie_db_username --env POSTGRES_PASSWORD=movie_db_password --env POSTGRES_DB=movie_db_dev postgres:12.1-alpine
                    docker run -d -p 8001:8000 --name movie-service --rm --env DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev --env CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/  $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG 
                    docker run -d -p 8080:8080 --name nginx --rm nginx:latest
                    
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl http://localhost:8001/api/v1/movies/docs 
                    curl http://localhost:8002/api/v1/casts/docs
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deployment in dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp  exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app exam --values=values.yml --namespace dev
                '''
                }
            }

        }

stage('Deployment in qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp  exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app exam --values=values.yml --namespace qa
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app exam --values=values.yml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
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
                cp exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app exam --values=values.yml --namespace prod
                '''
                }
            }

        }

}
}