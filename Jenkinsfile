pipeline {

    agent any

    environment {

        DOCKER_IMAGE = 'bhaskardo/nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"

        DEPLOYMENT_NAME = 'nginx-deployment'
        CONTAINER_NAME = 'nginx'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/bhaskarh993-oss/nginx-jenkins-kind.git'
            }
        }


        stage('Test') {

            steps {

                sh '''
                    test -f Dockerfile
                    test -f index.html
                    test -f deployment.yaml
                    test -f service.yaml

                    echo "Required files are present"
                '''
            }
        }


        stage('Docker Build') {

            steps {

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }


        stage('Docker Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }


        stage('Push to Docker Hub') {

            steps {

                sh '''
                    docker push \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


            stage('Deploy to Kubernetes') {
    steps {
        sshagent(['kind-server']) {
            sh '''
                scp -o StrictHostKeyChecking=no \
                    deployment.yaml \
                    service.yaml \
                    ingress.yaml \
                    ubuntu@172.31.2.160:/home/ubuntu/

                ssh -o StrictHostKeyChecking=no \
                    ubuntu@172.31.2.160 "
                        sed -i 's/IMAGE_TAG/${BUILD_NUMBER}/g' /home/ubuntu/deployment.yaml

                        kubectl apply -f /home/ubuntu/deployment.yaml
                        kubectl apply -f /home/ubuntu/service.yaml
                        kubectl apply -f /home/ubuntu/ingress.yaml

                        kubectl rollout status deployment/java-app

                        kubectl get pods
                        kubectl get svc
                        kubectl get ingress
                    "
            '''
            }
        }
    }


      
    
}
