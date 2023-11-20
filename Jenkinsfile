pipeline {
  agent {
    label 'Jenkins-Agent'
  }
  tools {
    jdk 'Java17'
    maven 'Maven3'
  }
  environment {
    APP_NAME "register-app-pipeline"
    RELEASE = "1.0.0"
    DOCKER_USER = "umechand"
    DOCKER_PASS 'Bullet@123'
    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
  }
  stages {
    stage("Cleanup Workspace") {
      steps {
        cleanWs()
      }
    }
    stage("Checkout from SCM") {
      steps {
        git branch: 'main', credentialsId: 'github', url: 'https://github.com/umechand/registration-app'
      }
    }

    stage("Build Application") {
      steps {
        sh "mvn clean package"
      }
    }
    stage("Test Application") {
      steps {
        sh "mvn test"
      }
    }

    stage("SonarQube Analysis") {
      steps {
        script {
          withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
            sh "mvn sonar:sonar"
          }
        }
      }
    }
    stage("Quality Gate") {
      steps {
        script {
          waitForQualityGate abortpipeline: false, credentialsId: 'jenkins-sonarqube-token'
        }
      }

    }
    stage("Build & push Docker Image") {
      steps {
        script {
          docker.withRegistery('', DOCKER_PASS) {
            docker_image = docker.build "${IMAGE_NAME}"
          }

          docker.withRegistry('', DOCKER_PASS) {
            docker_image.push("${IMAGE_TAG}")
            docker_image.push('latest')
          }
        }
      }

    }
  }
}
