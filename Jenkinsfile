pipeline {
    agent any
    environment {
        // Global environment variable for SonarCloud token
        SONAR_TOKEN = credentials('sonar-santa') // Reference to the Jenkins credential ID for SonarCloud token
    }
    tools {
        jdk 'jdk11' // Default JDK for the application
        maven 'maven3'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/Danish-Ansarii/secretsanta-generator.git'
            }
        }
        stage('Verify Default Java Version') {
            steps {
                sh 'java -version'
            }
        }
        stage('Maven Clean and Compile (Java 11)') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('SonarCloud Scan (Java 17)') {
            tools {
                jdk 'jdk17' // Switch to Java 17 for SonarCloud
            }
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=danish-ansarii \
                        -Dsonar.organization=danish-ansarii \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
        stage('Build Package and Create Artifact (Java 11)') {
            tools {
                jdk 'jdk11' // Switch back to Java 11 for the app build
            }
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercred', toolName: 'docker') {
                        sh 'docker build -t santaapp:1.0 .'
                        sh 'docker tag santaapp:1.0 danish84464/santaapp:1.0'
                        sh 'docker push danish84464/santaapp:1.0'
                    }
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercreddy', toolName: 'docker') {
                        sh 'docker container run -d -p 8080:8080 danish84464/santaapp:1.0'
                    }
                }
            }
        }
    }
}
