pipeline {
    agent {
        label 'jen-agent'
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "ranjeeth3302"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github-CRED', url: 'https://github.com/RanjeethAcharya/DevProject.git'
            }
        }
        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    // Execute SonarQube analysis
                    withSonarQubeEnv(credentialsId: "sonarqube") {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    // Wait for SonarQube Quality Gate
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    // Run Trivy vulnerability scan
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
                }
            }
        }
        stage("Cleanup artifacts") {
            steps {
                script {
                    // Clean up Docker images
                    docker.withRegistry('', DOCKER_PASS) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").remove()
                        docker.image("${IMAGE_NAME}:latest").remove()
                    }
                }
            }
        }
        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-54-83-173-110.compute-1.amazonaws.com:8080/job/register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
    // post {
    //     failure {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                  mimeType: 'text/html',
    //                  to: "ranjeethacharya3302@gmail.com"
    //     }
    //     success {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                  mimeType: 'text/html',
    //                  to: "ranjeethacharya3302@gmail.com"
    //     }      
    // }
}
