// ------------------------- Copying repo in EC2 -------------------------

pipeline {
    agent any

    environment {
        EC2_HOST = 'ubuntu@13.233.100.29'
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
                sh "echo Workspace is ${WORKSPACE}"
                sh "ls -la ${WORKSPACE}"

                sshagent(['aws-ec2-cred']) {
                    sh """
                        scp -o StrictHostKeyChecking=no -r ${WORKSPACE}/* ${EC2_HOST}:${REMOTE_DIR}
                    """
                }
            }
        }

        stage('Install Dependencies on EC2') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        sudo apt update &&
                        sudo apt install -y python3-pip &&
                        pip3 install streamlit pytest
                        '
                    """
                }
            }
        }

        stage('Run Tests on EC2') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        pytest ${REMOTE_DIR}/test_calculator.py
                        '
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sshagent(['aws-ec2-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        nohup streamlit run ${REMOTE_DIR}/calculator.py --server.port 8501 &
                        '
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
