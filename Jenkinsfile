pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "bassdocker" // replace this with your docker-id
DOCKER_IMAGE = "datascientestfastapi"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG ./cast-service
                 docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG ./movie-service
                 docker build -t $DOCKER_ID/nginx-service:$DOCKER_TAG ./nginx-service
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8081:8000 --name cast-service $DOCKER_ID/cast-service:$DOCKER_TAG
                    docker run -d -p 8082:8000 --name movie-service $DOCKER_ID/movie-service:$DOCKER_TAG
                    docker run -d -p 8080:8080 --name nginx-service $DOCKER_ID/nginx-service:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl --header "Content-Type: application/json" --request POST --data '{"name":"test","nationality":"FR"}' http://localhost:8081/
                    curl --header "Content-Type: application/json" --request POST --data '{"plot":"test","genres":["comedy","action"] ,"casts_id":[1,2,3]}' http://localhost:8082/
                    curl localhost:8080
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
                docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                docker push $DOCKER_ID/nginx-service:$DOCKER_TAG
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
                cat $KUBECONFIG > .kube/config
                helm upgrade --install cast-app charts/ --values=cast-service/values.yml --namespace dev --set image.repository="$DOCKER_ID/cast-service:$DOCKER_TAG"
                helm upgrade --install cast-app charts/ --values=movie-service/values.yml --namespace dev --set image.repository="$DOCKER_ID/movie-service:$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yml --namespace dev --set image.repository="$DOCKER_ID/nginx-service:$DOCKER_TAG"

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
                cat $KUBECONFIG > .kube/config
                helm upgrade --install cast-app charts/ --values=cast-service/values.yml --namespace staging --set image.repository="$DOCKER_ID/cast-service:$DOCKER_TAG"
                helm upgrade --install cast-app charts/ --values=movie-service/values.yml --namespace staging --set image.repository="$DOCKER_ID/movie-service:$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yml --namespace staging --set image.repository="$DOCKER_ID/nginx-service:$DOCKER_TAG"
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
                cat $KUBECONFIG > .kube/config
                helm upgrade --install cast-app charts/ --values=cast-service/values.yml --namespace prod --set image.repository="$DOCKER_ID/cast-service:$DOCKER_TAG"
                helm upgrade --install cast-app charts/ --values=movie-service/values.yml --namespace prod --set image.repository="$DOCKER_ID/movie-service:$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yml --namespace prod --set image.repository="$DOCKER_ID/nginx-service:$DOCKER_TAG"
                '''
                }
            }

        }

}

post { // send email when the job has failed
      
        failure {
            echo "This will run if the job failed"
            mail to: "bassdatascientest@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
       
    }
}
