pipeline {
    agent any

    environment {
        APP_IMAGE_NAME      = 'jowe2114/new-app'   //DockerHubub_repo/Image_name
        Dockerfile_PATH     = './Dockerfile'	       //Path to Dockerfile in github repo
        DEPLOYMENT_PATH     = './deployment.yaml'     //Path to deployment.yaml file in github repo
    }

    stages {
        
        stage('Build and Push to DockerHub') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${APP_IMAGE_NAME}:${BUILD_NUMBER} -f ${Dockerfile_PATH} ."

                    // Log in to DockerHub 
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u \$DOCKERHUB_USERNAME -p \$DOCKERHUB_PASSWORD"
                    }

                    // Push image to Docker Hub
                    sh "docker push ${APP_IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Remove Local Images') {
            steps {
                // Delete local Docker image after push them to DockerHub
                sh "docker rmi ${APP_IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Update Deployment Manifest and Deploy to OpenShift') {
            steps {
                script {
                    // Update deployment.yaml file with the new Docker image
                    sh "sed -i 's|image:.*|image: ${APP_IMAGE_NAME}:${BUILD_NUMBER}|g' ${DEPLOYMENT_PATH}"

                    // Deploy updated manifest to OpenShift
                   withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                        sh "export KUBECONFIG=\$KUBECONFIG_FILE && oc apply -f ${DEPLOYMENT_PATH} -n ibrahim"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}

