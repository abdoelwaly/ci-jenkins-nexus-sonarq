#project pipeline jenkins project jpro App

def COLOR_MAP = [           # Slack notification colors
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "MAVEN3.9.9"
        jdk "JDK17"
    }
    
    environment {          # Environment variables for Nexus Repository settings.xml
        SNAP_REPO = 'jpro-snapshot'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'Nexuspassword' # Chage it to Password for Nexus Repository
		RELEASE_REPO = 'jpro-release'
		CENTRAL_REPO = 'jpro-maven-central'
		NEXUSIP = '172.31.43.144' # ip address of Nexus Repository Server EC2 should be private ip
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'jpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin' # Credential name for Nexus Repository in Jenkins
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Archive Success'
                    archiveArtifacts artifacts: 'target/*.war'
                }
                failure {
                    echo 'Archive Failed'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {            # SonarQube Scanner documentation example
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=jprofile \
                   -Dsonar.projectName=jprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }  
        }
                stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("UploadArtifact"){     # google jenkins nexus artifact uploade  https://plugins.jenkins.io/nexus-artifact-uploader/
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'jproapp',
                     classifier: '',
                     file: 'target/jprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    }
    post {       # Post-build actions
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}