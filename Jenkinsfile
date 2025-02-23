pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'Java21'
        maven 'Maven3'
    }
    environment{
        APP_NAME = "ci-cd-pipeline-project"
        RELEASE = "1.0.0"
        DOCKER_USER = "mykulle"
        DOCKER_PASSWD = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Mykulle/CI-CD-Pipeline-Project'
            }
        }

        stage("Build application"){
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
                withSonarQubeEnv(installationName: 'sonarqube-scanner', credentialsId: 'jenkins-sonarqube-token') {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage("Quality Gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build and Push Docker Image"){
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASSWD){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    
                    docker.withRegistry('', DOCKER_PASSWD) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }

                }
            }
        }

        stage("Trigger CICD Pipeline"){
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkinsdevdman.click/job/gitops-ci-cd-pipeline/buildWithParameters?token=gitops-token'" 
                }
            }
        }
    }
}