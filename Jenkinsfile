pipeline {
    agent { label 'jen-agent' }
    tools {
        jdk 'java17'
        maven 'Maven3'
    }
    stages ('cleanup workspace') {
        steps {
            cleanWs()
        }
    }
    stages ('Checkout from SCM') {
        steps {
            git branch: 'main', credentialsId: 'github-CRED', url: 'https://github.com/RanjeethAcharya/DevProject.git'
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
}
