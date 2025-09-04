pipeline {
    agent any

    parameters {
        choice(
            choices: ['i-01f8cfb9e599102c9', 'i-0d37cafbc2530ce37']
        )
    }

    environment {
        DEPLOY_DIR = "/rocky"
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
                    def servers = [
                        server1: [user: 'ec2-user', ip: '172.31.6.203'],
                        server2: [user: 'root', ip: '192.168.1.50']
                    ]

                    def target = servers[params.SERVER_ID]

                    if (target == null) {
                        error "Unknown SERVER_ID: ${params.SERVER_ID}"
                    }

                    echo "Deploying to ${params.SERVER_ID} at ${target.ip}"
                    sh """
                        ssh -o StrictHostKeyChecking=no ${target.user}@${target.ip} "mkdir -p ${DEPLOY_DIR}"
                        scp -o StrictHostKeyChecking=no -r * ${target.user}@${target.ip}:${DEPLOY_DIR}/
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

