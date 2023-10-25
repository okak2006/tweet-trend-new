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
                echo "----------build started----------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------build ended----------------"
            }
        }
        stage("test") {
            steps {
                echo "----------unit test started----------------"
                sh 'mvn surefire-report:report'
                echo "----------unit test ended----------------"
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
        stage("Quality Gate"){
            steps {
                scripts {
                    timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate() 
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                    }
                }
            }
        }
    }
}
