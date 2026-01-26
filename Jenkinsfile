pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: '', description: 'Enter branch name manually')
    }

    stages {

        stage('Validate Branch & Checkout') {
            steps {
                script {
                    def repoUrl = "https://github.com/SurajHalakati/my-jenkins-project.git"

                    echo "Branch entered: ${params.BRANCH_NAME}"

                    try {
                        checkout([$class: 'GitSCM',
                            branches: [[name: "*/${params.BRANCH_NAME}"]],
                            userRemoteConfigs: [[url: repoUrl]]
                        ])

                        echo "✅ Branch is correct. Checkout successful!"
                    }
                    catch (err) {
                        error "❌ Branch is WRONG or not found: ${params.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Success') {
            steps {
                echo "✅ Pipeline Passed"
            }
        }

       stage('Validate Files') {
    steps {
        script {
            echo "✅ Validating allowed files..."

            def invalidFiles = sh(
                script: """
                find . -type f \
                ! -name '*.json' \
                ! -name 'Jenkinsfile' \
                ! -path './.git/*'
                """,
                returnStdout: true
            ).trim()

            echo "📌 Invalid files output:\n${invalidFiles}"

            if (invalidFiles) {
                error "❌ Invalid files found:\n${invalidFiles}\n\n✅ Allowed: only .json + Jenkinsfile"
            } else {
                echo "✅ All files are valid!"
            }
        }
    }
}
