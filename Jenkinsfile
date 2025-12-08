pipeline {
  agent any
  environment {
    IMAGE = "yourdockerhubuser/devops-capstone-app:${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Install') { steps { sh 'npm ci' } }
    stage('Build Image') { steps { sh "docker build -t ${IMAGE} ." } }
    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
          sh "docker push ${IMAGE}"
        }
      }
    }
    stage('Deploy') {
      steps {
        sh """
          docker stop capstone || true
          docker rm capstone || true
          docker pull ${IMAGE}
          docker run -d --name capstone -p 3000:3000 --restart unless-stopped ${IMAGE}
        """
      }
    }
  }
  post { always { sh 'docker image prune -f || true' } }
}
