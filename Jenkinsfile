pipeline {
    agent {
        node {
            label 'mavern'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
    }
    
    stages {
        stage("build") {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('SonarQube analysis') {
            environment {
                def scannerHome = tool 'sonarqube-scanner'; // make sure the name matches scanner tool name in Jenkins
            }
            steps {
                 withSonarQubeEnv('sonarqube-server') { // make sure the name matches sonarqube server name in Jenkins
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
