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
                    def DEPLOY_DIR = "/rocky"

                    echo "Deploying locally on Jenkins server into ${DEPLOY_DIR}..."

                    sh """
                        mkdir -p ${DEPLOY_DIR}
                        cp hello_world.py test_hello_world.py output.txt ${DEPLOY_DIR}/
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

