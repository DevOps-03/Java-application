pipeline {
    agent any

    tools {
        jdk 'Java21'
        maven 'Maven3'
    }

    environment {
        APP_NAME = 'hello-java'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // Change these values to match your JFrog Artifactory setup.
        JFROG_REGISTRY = 'mycompany.jfrog.io'
        JFROG_REPO = 'docker-local'
        JFROG_IMAGE = "${JFROG_REGISTRY}/${JFROG_REPO}/${APP_NAME}"

        // Jenkins credentials:
        // JFROG_CREDENTIALS = Username/Password credential
        // SONAR_TOKEN = Secret Text credential
        JFROG_CREDENTIALS = credentials('jfrog-docker-creds')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        sh '''
                            mvn clean verify sonar:sonar                               -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build                       -t ${JFROG_IMAGE}:${IMAGE_TAG}                       -t ${JFROG_IMAGE}:latest                       .
                '''
            }
        }

        stage('Push to JFrog Artifactory') {
            steps {
                sh '''
                    echo "$JFROG_CREDENTIALS_PSW" | docker login ${JFROG_REGISTRY}                       -u "$JFROG_CREDENTIALS_USR"                       --password-stdin

                    docker push ${JFROG_IMAGE}:${IMAGE_TAG}
                    docker push ${JFROG_IMAGE}:latest

                    docker logout ${JFROG_REGISTRY}
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
            junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
        }

        success {
            echo "Build ${BUILD_NUMBER} completed successfully."
            echo "Docker image: ${JFROG_IMAGE}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed. Check the failed stage and Jenkins console log."
        }
    }
}
