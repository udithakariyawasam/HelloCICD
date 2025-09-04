pipeline {
    agent any

    parameters {
        string(name: 'INSTANCE_ID', defaultValue: 'i-01f8cfb9e599102c9', description: 'Target EC2 instance ID')
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

        stage('Deploy') {
            steps {
                script {
                    // Fetch private IP of the EC2 instance
                    def SERVER_IP = sh(
                        script: """
                            aws ec2 describe-instances \
                                --instance-ids ${params.INSTANCE_ID} \
                                --query 'Reservations[0].Instances[0].PrivateIpAddress' \
                                --output text
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Deploying to EC2 instance ${params.INSTANCE_ID} with private IP ${SERVER_IP}..."

                    // Copy files into /rocky/ on the target server
                    sshagent (credentials: ['ec2-ssh-key']) {
                        sh """
                            DEPLOY_DIR="/rocky"
                            ssh -o StrictHostKeyChecking=no ec2-user@${SERVER_IP} "mkdir -p \$DEPLOY_DIR"
                            scp -o StrictHostKeyChecking=no hello_world.py test_hello_world.py output.txt ec2-user@${SERVER_IP}:\$DEPLOY_DIR/
                        """
                    }
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

