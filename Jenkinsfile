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
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
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
                   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image koushikn23/java-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
             }
        }

        stage("Cleanup Artifacts"){
             steps{
                script{
                   sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                   sh "docker rmi ${IMAGE_NAME}:latest"
               }
           }
        }
        stage("Trigger Deployment") {
             steps {
                script {
                   withCredentials([string(credentialsId: 'JENKINS_API_TOKEN', variable: 'API_TOKEN')]) {
                      sh """"
                       curl -v -k --user kumar:$API_TOKEN \
                      -X POST \
                      -H 'cache-control: no-cache' \
                      -H 'content-type: application/x-www-form-urlencoded' \
                      --data "IMAGE_TAG=${IMAGE_TAG}" \
                      "http://ec2-100-24-20-37.compute-1.amazonaws.com:8080/job/gitops-cd/buildWithParameters?token=gitops-token&IMAGE_TAG=${IMAGE_TAG}"
                   """
                    }
                }
             }
        }         
    }
}
