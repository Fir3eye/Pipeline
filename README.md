# Basic To Advanced Pipeline 
## 1. Build the Image Pipeline👷‍♂️
![WorkPenguinGIF](https://github.com/Fir3eye/Pipeline/assets/93431222/4ea5688e-0dbc-4246-aec0-d34038cd3687)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        sh "docker images --format '{{.Repository}}:{{.Tag}}' | grep ${IMAGE_NAME} | grep -v ${RELEASE}-${BUILD_NUMBER} | grep -v latest | xargs -I {} docker rmi {} || true" 
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
    }
}

```
## 2. Trivy File Scan Pipeline
![ScanGifScanningGIF (2)](https://github.com/Fir3eye/Pipeline/assets/93431222/bbaef868-172e-4a60-b6bf-242501acf3b2)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
			sh "docker images --format '{{.Repository}}:{{.Tag}}' | grep ${IMAGE_NAME} | grep -v ${RELEASE}-${BUILD_NUMBER} | grep -v latest | xargs -I {} docker rmi {} || true" 
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh "trivy image ${docker_image.id} > trivy.txt"
                }
            }
        }
    }
}

```
## 3. Push Image on Docker Hub Pipeline
![PushGuhanGuhanAmittGIF](https://github.com/Fir3eye/Pipeline/assets/93431222/d71ddf5a-906b-42ad-b892-4d72112b025d)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
			sh "docker images --format '{{.Repository}}:{{.Tag}}' | grep ${IMAGE_NAME} | grep -v ${RELEASE}-${BUILD_NUMBER} | grep -v latest | xargs -I {} docker rmi {} || true" 
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh "trivy image ${docker_image.id} > trivy.txt"
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }                     
    }
}

```
## 4. Clean Artifact Delete Docker image and Container Pipeline
![CleanCleaningGIF](https://github.com/Fir3eye/Pipeline/assets/93431222/948670f6-627e-400b-9cf6-dea6d8f95328)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh "trivy image ${docker_image.id} > trivy.txt"
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}



```
## 5. SonarQube Analysis Pipeline 
![KowalskiAnalysisKowalskiGIF](https://github.com/Fir3eye/Pipeline/assets/93431222/9b648b32-7e8c-4a15-8d76-f45b71c244e7)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh "trivy image ${docker_image.id} > trivy.txt"
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
    }
}

```
## 6. Quality Gate Pipeline 
![AutomaticGateBenincaGIF](https://github.com/Fir3eye/Pipeline/assets/93431222/d188737b-6fac-4f99-ac00-424519d79cf5)

```
pipeline{
    agent any

    environment {
        APP_NAME = "image_name"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_01_docker_push_img.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh "trivy image ${docker_image.id} > trivy.txt"
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
    }
}

```
