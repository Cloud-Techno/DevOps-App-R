pipeline {
  agent any 
  tools {
    jdk "jdk17"
    nodejs "node16"
  } 
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
    APP_NAME = "web-app-pipeline"
    RELEASE = "1.0.0"
    DOCKER_USER = "fates41"
    DOCKER_PASS = credentials('DOCKERHUB_PASSWORD_ID') // Güvenli kullanım
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
  }

  stages {
    stage ("clean workspace") {
      steps {
        cleanWs()
      }
    }

    stage('Checkout from Git') {
      steps {
        git branch: 'main', url: 'https://github.com/Cloud-Techno/DevOps-App-R.git'
      }
    }

    stage("Sonarqube Analysis") {
      steps {
        withSonarQubeEnv('SonarQube-Server') {
          sh '''$SCANNER_HOME/bin/sonar-scanner \
            -Dsonar.projectKey=web-app \
            -Dsonar.sources=.'''
        }
      }
    }

    stage("Quality Gate") {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
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
  }
}
