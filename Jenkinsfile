pipeline {
    agent any
    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage('Installation') {
            steps {
                sh "pip install -r requirements.txt"
            }
        }
        stage('Testing Env') {
            steps {
                sh "pytest"
            }
        }
        stage('Package to Codes') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
            }
        }
        stage('Deployment to Production') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "ssh-key", keyFileVariable: "MY_SSH_KEY", username: "username")]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        source app/venv/bin/activate
                        cd /home/ec2-user/app/
                        pip install -r requirements.txt
                        sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
    }
}