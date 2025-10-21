pipeline {
    agent any

    environment {
        APP_NAME = 'train-ticket'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/MachaSreenath/train-ticket-reservation.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building project using Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo '🚀 Deploying WAR to Tomcat...'
                withCredentials([
                    string(credentialsId: 'tomcat-user', variable: 'TOMCAT_USER'),
                    string(credentialsId: 'tomcat-pass', variable: 'TOMCAT_PASS'),
                    string(credentialsId: 'tomcat-host', variable: 'TOMCAT_HOST')
                ]) {
                    script {
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "Deploying ${warFile}..."
                        sh """
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \\
                            --upload-file ${warFile} \\
                            "$TOMCAT_HOST/manager/text/deploy?path=/${APP_NAME}&update=true"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Check Jenkins logs.'
        }
    }
}
