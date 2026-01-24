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
                bat "dir"
            }
        }

        stage("Validate JSON Syntax") {
            steps {
                echo "🔍 Validating all JSON files..."
                bat '''
                for /r %%f in (*.json) do (
                    echo Checking JSON: %%f
                    python -m json.tool < "%%f" > nul
                    if errorlevel 1 exit /b 1
                )
                '''
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
