pipeline {
    agent any

    environment {
        APP_IMAGE_NAME      = 'ibrahimadel10/new-app'   //github_repo/image_name
        Dockerfile_PATH     = './Dockerfile'	       //path to Dockerfile in github repo
        DEPLOYMENT_PATH     = './deployment.yaml'     //path to deployment file in github repo
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
                // Delete local Docker image
                sh "docker rmi ${APP_IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Update Deployment Manifest and Deploy to OpenShift') {
            steps {
                script {
                    // Update deployment.yaml with new Docker Hub image
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

