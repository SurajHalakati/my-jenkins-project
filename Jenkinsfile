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

        stage("Build (Create ZIP)") {
            steps {
                echo "📦 Creating ZIP artifact..."
                bat '''
                if not exist output mkdir output
                powershell Compress-Archive -Path * -DestinationPath output\\arm_templates.zip -Force
                '''
            }
        }

        stage("Archive Artifact") {
            steps {
                archiveArtifacts artifacts: "output/*.zip", fingerprint: true
                echo "✅ Artifact archived in Jenkins"
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
