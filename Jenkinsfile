def dockerImage
pipeline {
    agent {
        label 'slave-agent'
    }
    environment {
        IMAGE_NAME = "afeez1131/java-project"
        REGISTRY_URL = 'https://docker.io'
        CREDS_ID     = 'dockerHubCredential'
    }
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK17'
    }
    stages {
        // stage git checkout
        stage('Fetch from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Afeez1131/java-project.git'
            }
        }

        // compile
        stage('Compile Code'){
            steps {
                sh 'mvn clean compile'
            }
        }
        // Test
        stage('Test Code'){
            steps {
                sh 'echo "running test"'
                sh 'mvn test'
            }
        }
        
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Checkstyle analysis completed'
                }
                failure {
                    echo 'Checkstyle analysis failed'
                }
            }
        }

        stage('Sonarqube analysis') {
            environment {
                SCANNER_HOME = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${SCANNER_HOME}/bin/sonar-scanner \
                      -Dsonar.projectKey=java-project \
                      -Dsonar.projectName=java-project \
                      -Dsonar.projectVersion=1.0 \
                      -Dsonar.sources=src/ \
                      -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                      -Dsonar.junit.reportsPath=target/surefire-reports/ \
                      -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                      -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        // package
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
            environment {
                NEXUS_URL = '172.31.40.220:8081'
                NEXUS_CREDENTIALS_ID = 'NexusCredentials'
                NEXUS_REPOSITORY = 'java-project'
                NEXUS_GROUP_ID = 'com.visualpathit'
                NEXUS_ARTIFACT_ID = 'java-project'
                NEXUS_ARTIFACT_VERSION = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP.replaceAll('[: ]', '_')}"
                NEXUS_ARTIFACT_FILE = 'target/vprofile-v2.war'
                NEXUS_ARTIFACT_TYPE = 'war'
            }
            steps {
                echo "Uploading artifact to Nexus: ${NEXUS_ARTIFACT_FILE}"
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: NEXUS_URL,
                    groupId: NEXUS_GROUP_ID,
                    version: NEXUS_ARTIFACT_VERSION,
                    repository: NEXUS_REPOSITORY,
                    credentialsId: NEXUS_CREDENTIALS_ID,
                    artifacts: [
                        [
                            artifactId: NEXUS_ARTIFACT_ID,
                            classifier: '',
                            file: NEXUS_ARTIFACT_FILE,
                            type: NEXUS_ARTIFACT_TYPE
                        ]
                    ]
                )
            }
        }
        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: CREDS_ID,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }
}
