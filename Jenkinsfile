pipeline {
  agent any

  environment {
    APP_NAME = 'train-ticket'           
    TOMCAT_HOST = '54.242.124.202'   
    TOMCAT_PORT = '8081'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'git@github.com:MachaSreenath/train-ticket-reservation.git', credentialsId: 'github-ssh'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Deploy to Tomcat') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'tomcat-deployer', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
          script {
            def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
            echo "Deploying ${warFile} to http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/${APP_NAME}&update=true"
            sh """
              curl --fail --silent --show-error --upload-file ${warFile} \\
                   "http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/${APP_NAME}&update=true" \\
                   --user "${TOMCAT_USER}:${TOMCAT_PASS}"
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deployment successful!'
    }
    failure {
      echo '❌ Deployment failed!'
    }
  }
}
