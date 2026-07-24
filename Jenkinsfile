pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/apps/pam-server"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    python -m pip install --upgrade pip
                    pip install pytest
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest test_backup.py
                '''
            }
        }

        stage('Package Application') {
            steps {
                sh '''
                    tar -czf deployment.tar.gz .
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no deployment.tar.gz ubuntu@54.92.175.209:$APP_DIR/

                        ssh -o StrictHostKeyChecking=no ubuntu@54.92.175.209 << EOF

                        cd $APP_DIR

                        tar -xzf deployment.tar.gz

                        python3 run_all.py

                        EOF
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh '''
                        ssh ubuntu@54.92.175.209 "ls -lah $APP_DIR"
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }

        always {
            cleanWs()
        }

    }
}


