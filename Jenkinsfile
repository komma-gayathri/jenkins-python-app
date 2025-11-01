pipeline {
    agent any

    environment {
        // change this to your EC2 public IP
        TARGET_SERVER = "3.7.70.50"
        SSH_KEY = "ecs-key.pem"
        SSH_USER = "ubuntu"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/komma-gayathri/jenkins-python-app.git'
            }
        }

        stage('Build App') {
            steps {
                echo '🔧 Building Python app...'
                sh 'python3 --version'
                sh 'echo "Build successful!"'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '🚀 Deploying to EC2 instance...'
                // copy app files to EC2
                sh '''
                ssh -o StrictHostKeyChecking=no -i ~/.ssh/${SSH_KEY} ${SSH_USER}@${TARGET_SERVER} '
                    sudo rm -rf ~/app &&
                    mkdir ~/app'
                scp -o StrictHostKeyChecking=no -i ~/.ssh/${SSH_KEY} -r * ${SSH_USER}@${TARGET_SERVER}:~/app
                '''
            }
        }

        stage('Restart App') {
            steps {
                echo '♻️ Restarting Python app...'
                sh '''
                ssh -i ~/.ssh/${SSH_KEY} ${SSH_USER}@${TARGET_SERVER} '
                    pkill -f app.py || true
                    nohup python3 ~/app/app.py > app.log 2>&1 &
                '
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build Success',
                 body: 'The Python app has been deployed successfully.'
        }
        failure {
            echo '❌ Deployment Failed!'
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build Failed',
                 body: 'The Jenkins pipeline failed. Check console output.'
        }
    }
}
