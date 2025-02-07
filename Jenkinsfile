#Build the project from Nexus Repository

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
		NEXUSIP = '172.31.43.144' # ip address of Nexus Repository Server EC2
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
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }  
        }    
    }
}