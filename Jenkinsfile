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
                 docker rm -f cast-service
                 docker rm -f movie-service
                 docker rm -f nginx
                 docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG ./cast-service
                 docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG ./movie-service
                 docker build -t $DOCKER_ID/nginx:$DOCKER_TAG ./nginx
                sleep 6
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
                docker push $DOCKER_ID/nginx:$DOCKER_TAG
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
                helm upgrade --install cast-app charts/ --values=cast-service/values.yaml --namespace dev --set image.repository="$DOCKER_ID/cast-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install movie-app charts/ --values=movie-service/values.yaml --namespace dev --set image.repository="$DOCKER_ID/movie-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yaml --namespace dev --set image.repository="$DOCKER_ID/nginx" --set image.tag="$DOCKER_TAG"

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
                helm upgrade --install cast-app charts/ --values=cast-service/values.yaml --namespace staging --set image.repository="$DOCKER_ID/cast-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install movie-app charts/ --values=movie-service/values.yaml --namespace staging --set image.repository="$DOCKER_ID/movie-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yaml --namespace staging --set image.repository="$DOCKER_ID/nginx" --set image.tag="$DOCKER_TAG"
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
                helm upgrade --install cast-app charts/ --values=cast-service/values.yaml --namespace prod --set image.repository="$DOCKER_ID/cast-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install movie-app charts/ --values=movie-service/values.yaml --namespace prod --set image.repository="$DOCKER_ID/movie-service" --set image.tag="$DOCKER_TAG"
                helm upgrade --install nginx-app charts/ --values=nginx/values.yaml --namespace prod --set image.repository="$DOCKER_ID/nginx" --set image.tag="$DOCKER_TAG"
                '''
                }
            }

        }

}
}
