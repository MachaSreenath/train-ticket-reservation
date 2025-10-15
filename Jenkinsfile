pipeline {
    agent any

    environment {
        SERVER = 'ubuntu@ec2-13-222-133-81.compute-1.amazonaws.com'
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
                sshagent(['ssh-key']) {
                    sh '''
                        echo "Stopping old app..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} "pkill -f '.jar' || true"
                        echo "Copying new app..."
                        scp -o StrictHostKeyChecking=no target/*.jar ${SERVER}:/home/ubuntu/
                        echo "Starting app..."
                        ssh -o StrictHostKeyChecking=no ${SERVER} "nohup java -jar /home/ubunut/*.jar > app.log 2>&1 &"
                    '''
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
