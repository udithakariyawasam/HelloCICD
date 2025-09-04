pipeline {
    agent any

    parameters {
        choice(name: 'INSTANCE_ID', choices: ['i-01f8cfb9e599102c9', 'i-0d37cafbc2530ce37'], description: 'Select EC2 Instance ID to deploy to')
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
                echo "Fetching IP for instance ${params.INSTANCE_ID}..."
                // Requires AWS CLI configured on Jenkins
                sh '''
                    SERVER_IP=$(aws ec2 describe-instances \
                        --instance-ids ${INSTANCE_ID} \
                        --query "Reservations[0].Instances[0].PublicIpAddress" \
                        --output text)
                    echo "SERVER_IP=$SERVER_IP" > server_info.env
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to EC2 instance ${params.INSTANCE_ID}..."
                sh '''
                    source server_info.env
                    SERVER_USER=ec2-user
                    DEPLOY_DIR="/rocky"

                    ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "mkdir -p $DEPLOY_DIR"
                    scp -o StrictHostKeyChecking=no -r * $SERVER_USER@$SERVER_IP:$DEPLOY_DIR/
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'output.txt', followSymlinks: false
        }
    }
}

