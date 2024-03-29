def imageName="192.168.44.44:8082/docker-local/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""


pipeline {
    agent {
        label 'agent'
    }
    environment {
        scannerHome = tool 'SonarQube'
    }
    stages {
        stage('Clone Frontend Repository') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            steps {
                sh '''pip3 install -r requirements.txt
python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'''
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(1) {
                    waitForQualityGate true
                }
            }
        }
        stage('Build Application') {
            steps {
                script {
                    dockerTag = "RC-${BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag", ".")
                }
            }
        }
        stage('Deploy Image') {
            steps {
                withDockerRegistry(credentialsId: "${registryCredentials}", url: "${dockerRegistry}") {
                    script {
                        applicationImage.push()
                        applicationImage.push("latest")
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: 'test-results/*.xml'
            cleanWs()
        }
    }
}