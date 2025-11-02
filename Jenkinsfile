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
                ssh -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/ecs-key.pem ubuntu@3.7.70.50 "
                    rm -rf ~/app &&
                    mkdir ~/app
                "
                scp -o StrictHostKeyChecking=no -i /var/jenkins_home/.ssh/ecs-key.pem -r app.py requirements.txt ubuntu@3.7.70.50:~/app
                '''
            }
        }

        stage('Restart App') {
            steps {
                echo "♻️ Restarting Python app..."
                sh '''
                ssh -i /var/jenkins_home/.ssh/ecs-key.pem ubuntu@3.7.70.50 "
                    cd ~/app &&
                    python3 -m venv venv &&
                    source venv/bin/activate &&
                    pip install --upgrade pip &&
                    pip install flask &&
                    pkill -f app.py || true &&
                    nohup venv/bin/python app.py > app.log 2>&1 &
                "
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
            emailext(
                to: 'gayathrikomma309@gmail.com',
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Hi Gayathri,

                The build for job '${env.JOB_NAME}' (build #${env.BUILD_NUMBER}) succeeded!
                Check details at: ${env.BUILD_URL}

                Regards,
                Jenkins
                """
            )
        }
        failure {
            echo "❌ Deployment failed!"
            emailext(
                to: 'gayathrikomma309@gmail.com',
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Hi Gayathri,

                The build for job '${env.JOB_NAME}' (build #${env.BUILD_NUMBER}) failed.
                Check the console output for details: ${env.BUILD_URL}

                Regards,
                Jenkins
                """
            )
        }
    }
}
