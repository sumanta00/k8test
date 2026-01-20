pipeline {
  agent any

  tools {
    jdk 'JDK21'
    maven 'Maven'
  }

  environment {
    DOCKER_USER = "sumanta00"
    IMAGE_NAME = "indiaproj"
    IMAGE_TAG = "1.0"
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            credentialsId: 'mygithubcred',
            url: 'https://github.com/sumanta00/k8test.git'
      }
    }

    stage('Run Tests') {
      steps {
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
        bat 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        bat """
        docker build -t %DOCKER_USER%/%IMAGE_NAME%:%IMAGE_TAG% .
        """
      }
    }

    stage('Push Docker Image to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhubpwd', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
          docker login -u %USER% -p %PASS%
          docker push sumanta00/indiaproj:1.0
          """
        }
      }
    }

    stage('Deploy Project to K8s') {
      steps {
        bat """
        minikube delete
        minikube start
        minikube image load sumanta00/indiaproj:1.0
        kubectl apply -f deployment.yaml
        kubectl apply -f services.yaml
        kubectl get pods
        kubectl get services
        minikube addons enable dashboard
        minikube dashboard
        """
      }
    }
  }

  post {
    success {
      echo 'Pipeline Executed Successfully!'
    }
    failure {
      echo 'Pipeline Failed — Check Logs'
    }
  }
}
