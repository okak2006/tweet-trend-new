def registry = 'https://okak2006.jfrog.io/'
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
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifactory"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                            "pattern": "jarstaging/(*)",
                            "target": "libs-release-local/{1}",
                            "flat": "false",
                            "props" : "${properties}",
                            "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
                }
            }   
        }
    }
}
