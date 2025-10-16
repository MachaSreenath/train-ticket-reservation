pipeline {
    agent any

    environment {
        SERVER = 'ubuntu@ec2-54-209-242-126.compute-1.amazonaws.com'
        DEPLOY_PATH = '/home/ubuntu/app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/MachaSreenath/train-ticket-reservation.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Approval Before Deploy') {
            steps {
                script {
                    def userInput = input(
                        id: 'DeployApproval', 
                        message: 'Do you want to deploy this build to the server?',
                        parameters: [choice(name: 'Approve', choices: 'No\nYes', description: 'Approve deployment')]
                    )
                    if (userInput == 'No') {
                        error("Deployment aborted by user")
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['jenkins-ssh-key']) {
                    sh """
                        echo "Stopping old app..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} 'pm2 delete app || true'
                        
                        echo "Creating deploy directory if not exists..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} 'mkdir -p ${DEPLOY_PATH}'
                        
                        echo "Copying new app files..."
                        scp -o StrictHostKeyChecking=no -r * ${SERVER}:${DEPLOY_PATH}/
                        
                        echo "Installing Node.js dependencies..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} 'cd ${DEPLOY_PATH} && npm install'
                        
                        echo "Starting app using pm2..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} 'cd ${DEPLOY_PATH} && pm2 start index.js --name app'
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed or Aborted'
        }
    }
}
