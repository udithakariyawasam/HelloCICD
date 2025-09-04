pipeline {
    agent any

    parameters {
        choice(name: 'INSTANCE_ID', choices: ['i-01f8cfb9e599102c9'], description: 'Select EC2 Instance ID to deploy to')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running Hello World script...'
                sh 'python3 hello_world.py > output.txt'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'pytest --maxfail=1 --disable-warnings -q'
            }
        }

        stage('Get Server IP') {
            steps {
                script {
                    echo "Fetching IP for instance ${params.INSTANCE_ID}..."
                    def ip = sh(
                        script: "aws ec2 describe-instances --instance-ids ${params.INSTANCE_ID} --query 'Reservations[0].Instances[0].PublicIpAddress' --output text",
                        returnStdout: true
                    ).trim()
                    echo "Found IP: ${ip}"
                    env.SERVER_IP = ip   // store into Jenkins environment
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying to EC2 instance ${params.INSTANCE_ID} with IP ${env.SERVER_IP}..."
                    sh """
                        SERVER_USER=ec2-user
                        DEPLOY_DIR="/rocky"
                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@${env.SERVER_IP} "mkdir -p \$DEPLOY_DIR"
                        scp -o StrictHostKeyChecking=no -r * \$SERVER_USER@${env.SERVER_IP}:\$DEPLOY_DIR/
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'output.txt', followSymlinks: false
        }
    }
}

