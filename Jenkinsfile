pipeline {
  agent any

  parameters {
    string(name: 'EC2_HOST', defaultValue: '1.2.3.4', description: 'Target EC2 Server IP or DNS')
  }

  environment {
    APP_NAME = 'logo-server'
    IMAGE = "shubhamsoni3332/${APP_NAME}:${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      steps {
        sh 'npm install'
        sh 'node -v'
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${IMAGE} ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDS',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
          sh "docker push ${IMAGE}"
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'EC2_SSH_KEY',
                                          keyFileVariable: 'KEY',
                                          usernameVariable: 'SSH_USER')]) {
          sh """
            ssh -o StrictHostKeyChecking=no -i $KEY $SSH_USER@${params.EC2_HOST} \
              'docker pull ${IMAGE} &&
               docker rm -f ${APP_NAME} || true &&
               docker run -d --restart unless-stopped -p 80:3000 --name ${APP_NAME} ${IMAGE}'
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ App deployed successfully on ${params.EC2_HOST}"
    }
    failure {
      echo "❌ Build failed, check logs!"
    }
  }
}
