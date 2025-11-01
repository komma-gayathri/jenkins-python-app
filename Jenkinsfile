pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/komma-gayathri/jenkins-python-app.git'
            }
        }

        stage('Build App') {
            steps {
                echo "🔧 Building Python app..."
                sh 'python3 --version'
                sh 'echo Build successful!'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "🚀 Deploying to EC2 instance..."
                sh '''
                ssh -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/ecs-key.pem ubuntu@3.7.70.50 '
                    sudo rm -rf ~/app &&
                    mkdir ~/app
                '
                scp -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/ecs-key.pem -r Jenkinsfile app.py requirements.txt ubuntu@3.7.70.50:~/app
                '''
            }
        }

        stage('Restart App') {
            steps {
                echo "♻️ Restarting Python app..."
                sh '''
                ssh -i /var/jenkins_home/.ssh/ecs-key.pem ubuntu@3.7.70.50 '
                    pkill -f app.py || true
                    nohup python3 ~/app/app.py > app.log 2>&1 &
                '
                '''
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
