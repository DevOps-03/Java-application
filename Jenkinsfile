pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/DevOps-03/Java-application.git'
            }
        }

        stage('Build and SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        mvn clean verify \
                        org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=hello-java
                    '''
                }
            }
        }

        stage('Create Docker Image') {
            steps {
                sh 'docker build -t bujji:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            -u "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker tag bujji:${BUILD_NUMBER} \
                            $DOCKERHUB_USERNAME/hello-java:${BUILD_NUMBER}

                        docker push \
                            $DOCKERHUB_USERNAME/hello-java:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }
}
