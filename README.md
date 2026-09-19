# Hello Java - Jenkins / SonarQube / Docker / JFrog

Simple Java 21 sample application for a Jenkins Declarative Pipeline.

## Java version

Tested/targeted for:

Java 21.0.12

The Maven compiler is configured with:

maven.compiler.release=21

## Repository structure

hello-java-jenkins/
├── Jenkinsfile
├── Dockerfile
├── .dockerignore
├── pom.xml
└── src/
    ├── main/java/com/example/HelloWorld.java
    └── test/java/com/example/HelloWorldTest.java

## Jenkins prerequisites

Install/configure:

1. JDK 21
2. Maven
3. Docker
4. SonarQube server
5. JFrog Artifactory Docker repository
6. Jenkins credentials:
   - sonar-token: Secret Text containing SonarQube token
   - jfrog-docker-creds: Username/Password for JFrog

Configure Jenkins Global Tool Configuration:

JDK name:
Java21

Maven name:
Maven3

Configure SonarQube in:
Manage Jenkins -> System -> SonarQube servers

Use the SonarQube installation name:
SonarQube

## JFrog values to change

Edit Jenkinsfile:

JFROG_REGISTRY = 'mycompany.jfrog.io'
JFROG_REPO = 'docker-local'

Example resulting image:

mycompany.jfrog.io/docker-local/hello-java:15

## Pipeline flow

Checkout
   |
SonarQube scan + Maven verify
   |
Maven build
   |
Docker image build
   |
Docker login
   |
Push image to JFrog Artifactory

## Run locally

mvn clean package

java -jar target/hello-java-1.0.0.jar

Build Docker image:

docker build -t hello-java:local .

Run:

docker run --rm hello-java:local
test123


