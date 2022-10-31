def imageName= "snuuffy/frontend"
def dockerRegistry = ""
def registryCredentials = "Dockerhub"
def dockerTag = ""

pipeline {
    agent {
  label 'agent'
    }
    
    environment {
  scannerHome = tool 'SonarQube'
    }


    stages {
        stage('Git Clone') {
            steps {
                checkout scm
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        stage('Push to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials"){
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}