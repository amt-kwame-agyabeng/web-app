def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]


pipeline {
    agent any
    tools{
        maven "maven"
    }
    
    stages {
        stage("clone repository") {
            steps {
                git branch: 'main', url: 'https://github.com/amt-kwame-agyabeng/web-app'
            }
            
        }
        stage("maven build") {
            steps {
                sh "mvn clean"
            }
        }
        stage("maven test") {
            steps {
                sh "mvn test"
            }
        }
        stage("maven package") {
            steps {
                sh "mvn package"
            }
        }
        stage("Sonarqube analysis") {
            environment {
                ScannerHone = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '${ScannerHome}/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar5.0/bin/sonar-scanner -Dsonar.projectKey=webapp-pipeline'
                    }
                }
            }
        }
        stage("deploy to nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/webapp-project-pipeline/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '54.81.139.142:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '4.1.0-RELEASE'
            }
        }
        stage("deploy to tomcat") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://3.84.236.148:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
        stage("quality gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
            
        }
        
    }
    
    post {
        success {
            slackSend channel: '#project', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: '#project', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }
    }
    
  
}
