pipeline{
     agent{
        label "jenkins-agent"
     }
     tools {
        jdk 'Java17'
        maven 'Maven3'
     }
     environment{
         APP_NAME = "complete-prodcution-e2e-pipeline"
         RELEASE = "1.0.0"
         DOCKER_USER = "maheswar443"
         DOCKER_PASS = 'dockerhub'
         IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
         IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
         JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
         
    }
     stages{
        stage("cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
     
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/maheswar443/complete-prodcution-e2e-pipeline.git'
            }
        }
        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }
        }
        stage("Sonarqube Analysis"){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                      sh "mvn sonar:sonar"
                      }
                }
            }
        }
        stage("Quality Gates"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build & Push Docker Image"){
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user maheswar:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://54.167.72.31:8080/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                }
            }

        }

        
     }
    

}          