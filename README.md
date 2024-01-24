# Jenkins Pipeline 

> This document provides an overview of the Jenkins pipeline for building and deploying Dockerized applications to OpenShift cluster.


## Pipeline Overview

The Jenkins pipeline follows these stages to build, push, and deploy Docker images to dockerhub and an OpenShift cluster:

1. **Build and Push to DockerHub:** Build and push image to DockerHub.

2. **Remove Images:** Remove local Docker image from the Jenkins server.

3. **Update Deployment Manifest and Deploy to OpenShift:** Update OpenShift deployment with the new image versions and deploy it to OpenShift cluster.

## Environment Variables

The following environment variables are used in the Jenkins pipeline:

- `APP_IMAGE_NAME`: github_repo/image_name.
- `Dockerfile_PATH`: path to Dockerfile in github repo.
- `DEPLOYMENT_PATH`: path to deployment file in github repo.
  

## Pipeline Steps

### Build and Push to DockerHub:

```
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
```



### Remove Local Images:

```
stage('Remove Local Images') {
            steps {
                // Delete local Docker image
                sh "docker rmi ${APP_IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

```

### Update Deployment Manifest and Deploy to OpenShift:

```
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
```

### Post-Build Actions
In case of pipeline success or failure, the following messages will be displayed:

```
post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
```
----
### - Successfully run the pipeline
![](https://github.com/IbrahimmAdel/Jenkins-Pipeline/blob/Master/screenshots/Screenshot%20from%202024-01-23%2015-51-23.png)
---

### - Deployment from OpenShift cluster
![](https://github.com/IbrahimmAdel/Jenkins-Pipeline/blob/Master/screenshots/Screenshot%20from%202024-01-23%2015-50-05.png)
