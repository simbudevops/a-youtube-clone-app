pipeline {
    agent any
    tools {
        jdk 'jdk21'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/simbudevops/a-youtube-clone-app.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=YOUTUBE-CICD \
                    -Dsonar.projectKey=youtube-cicd'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t youtube-clone ."
                        sh "docker tag youtube-clone simbudevops/youtube-clone:latest "
                        sh "docker push simbudevops/youtube-clone:latest "
                    }
                }
            }
        }
        stage("TRIVY") {
            steps {
                sh "trivy image simbudevops/youtube-clone:latest > trivyimage.txt"
            }
        }
    }
}
