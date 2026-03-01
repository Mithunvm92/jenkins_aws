pipeline {
    agent any

    environment {
        EC2_HOST = 'ubuntu@localhost'
        REMOTE_DIR = '/home/ubuntu'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Mithunvm92/jenkins_aws.git', branch: 'main'
            }
        }

        stage('Copy Application to EC2') {
            steps {
                echo 'Deploying to EC2...'

                sshagent(['aws-ec2-cred']) {
                    sh """
                        scp -o StrictHostKeyChecking=no -r ${WORKSPACE}/. ${EC2_HOST}:${REMOTE_DIR}
                    """
                }
            }
        }

        stage('Install Dependencies on EC2') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} "
                            sudo apt update &&
                            sudo apt install -y python3-pip &&
                            sudo pip3 install streamlit pytest
                        "
                    """
                }
            }
        }

        stage('Run Tests on EC2') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} "
                            cd ${REMOTE_DIR} &&
                            pytest test_calculator.py
                        "
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} "
                            cd ${REMOTE_DIR} &&
                            nohup streamlit run calculator.py --server.port 8501 > app.log 2>&1 &
                        "
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}

