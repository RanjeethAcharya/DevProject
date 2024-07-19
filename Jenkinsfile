pipeline {
    agent { label 'jen-agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages {
        stage('cleanup workspace') {
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
        stage("SonarQube analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: "jenkins-sonarqube-token") {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
    }
}
