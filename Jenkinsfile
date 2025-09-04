pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo "📥 Pulling code from GitHub..."
                checkout scm
            }
        }

        stage('Run Script') {
            steps {
                echo "🚀 Running Python script..."
                sh "python3 hello.py > output.txt"
            }
        }

        stage('Archive Results') {
            steps {
                echo "📦 Saving output..."
                archiveArtifacts artifacts: 'output.txt', followSymlinks: false
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline finished successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
