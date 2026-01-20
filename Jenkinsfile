pipeline {
  agent any

  tools {
    jdk 'JDK21'
    maven 'Maven'
  }

  environment {
    DOCKER_USER = "sumanta00"
  }

  stages {

    stage('Checkout Code') {
      steps {
        echo 'Pulling Code from Github'
        git branch: 'main', credentialsId: 'mygithubcred', url: 'https://github.com/sumanta00/k8test.git'
      }
    }

    stage('Run Tests') {
      steps {
        echo 'Executing JUnit Tests'
        bat 'mvn clean test'
      }
      post {
        always {
          junit '**/target/surefire-reports/*.xml'
        }
      }
    }

    stage('Build JAR') {
      steps {
        echo 'Packaging JAR File'
        bat 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo 'Building Docker Image'
        bat "docker build -t ${DOCKER_USER}/indiaproj:1.0 ."
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'DOCKER_PASS')]) {
          bat """
          echo %DOCKER_PASS% | docker login -u ${DOCKER_USER} --password-stdin
          docker push ${DOCKER_USER}/indiaproj:1.0
          """
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline Execution Completed Successfully!'
    }
    failure {
      echo 'Pipeline Failed — Check Logs'
    }
  }
}
