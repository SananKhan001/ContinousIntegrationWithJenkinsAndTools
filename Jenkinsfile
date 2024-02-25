pipeline {
    agent any 

    // Specifying tools to be used
    tools {
        maven "MAVEN3"
        jdk "OracleJDK17"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_IP = '10.0.13.28'
        NEXUS_PORT = '8081'
        NEXUS_GRP_REPO = 'vprofile-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSCANNER = 'sonarscanner'
        SONARSERVER = 'sonarserver'
    }

    stages {
        stage('Build') {
            steps {
                echo "Executing Build Stage"
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Build Success"
                    echo 'Now Archiving.'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Executing Test Stage'
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                echo 'Executing Checkstyle Analysis Stage'
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
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

        stage('Quality Gate') {
            steps {
                timeout(2) {
                    // some block 
                    waitForQualityGate abortPipeline: true 
                }
            }
        }

        stage('UploadArtifact') {
            steps {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}"
                        groupId: "QA",
                        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                        repository: "${RELEASE_REPO}",
                        credentialsId: "${NEXUS_LOGIN}",
                        artifacts: [
                            [artifactId: 'vproapp',
                            classifier: '',
                            file: 'target/vprofile-v1.war',
                            type: 'war']
                        ]
                    )
            }
        }
    }
}
