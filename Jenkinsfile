pipeline {
    agent { label 'jen-agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
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
    }
}
