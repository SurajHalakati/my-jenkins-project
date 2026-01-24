pipeline {
    agent any

    stages {
        stage("Checkout") {
            steps {
                echo "✅ Repo checked out successfully"
            }
        }

        stage("List Files") {
            steps {
                sh "ls -la"
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline SUCCESS"
        }
        failure {
            echo "❌ Pipeline FAILED - Check console output"
        }
    }
}
