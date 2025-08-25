pipeline {
    agent any
    environment {
        SERVER_IP = withCredentials('prod-server-ip')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/JamesZhao007/jenkins-project.git', branch: 'main'
                sh 'ls -ltr'
            }
        }
        stage('Setup') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'pytest'
            }
        }
        stage('Package Code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git'"
                sh 'ls -latr'
            }
        }
        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key',
                        keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    # Copy app zip to EC2 instance
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
                    # SSH into EC2 and deploy
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        source /home/ec2-user/app/venv/bin/activate
                        cd /home/ec2-user/app
                        pip install -r requirements.txt
                        sudo systemctl restart flaskapp.service
                    EOF
                    '''
                }
            }
        }
    }
}
