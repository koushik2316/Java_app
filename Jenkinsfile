pipeline {
    agent{
        label 'Jenkins-agent'
    }
    tools{
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "java-app-pipeline"
        RELEASE_VERSION = "1.0.0"
        DOCKER_USER = "koushikn23"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE_VERSION}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
             steps{
                cleanWs()
             }
        }

        stage("Checkout from SCM"){
             steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/koushik2316/Java_app.git'
             }
        }

        stage("Build Application"){
             steps{
                sh "mvn clean package"
             }
        }

        stage("Test Application"){
             steps{
                sh "mvn test"
             }
        }
        stage("SonarQube Analysis"){
             steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                       sh "mvn sonar:sonar"    
                   }
                }
             }
        }
        stage("Quality Gate"){
             steps{
                script{
                   waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
             }
        }
        stage("Build & Push Docker Image"){
             steps{
                script{
                   docker.withRegistry('',DOCKER_PASS) {
                       docker_image = docker.build "${IMAGE_NAME}"
                   }
                   docker.withRegistry('', DOCKER_PASS) {
                       docker_image.push("${IMAGE_TAG}")
                       docker_image.push("latest")
                   }
                }
             }
        }
        stage("trivy scan"){
             steps{
                script{
                   sh "trivy image --exit-code 1 --severity CRITICAL,HIGH ${IMAGE_NAME}:${IMAGE_TAG}"
                }
             }
        }
    
    }
}
