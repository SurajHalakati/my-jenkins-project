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
                        echo "❌ Actual error: ${err}"
                        error "❌ Branch is WRONG or not found: ${params.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Validate Files') {
            steps {
                script {
                    echo "✅ Validating allowed files..."

                    def invalidFiles = bat(
                        script: """
                        powershell -Command 'Get-ChildItem -Recurse -File | Where-Object { $_.Name -notmatch "\\.json$" -and $_.Name -ne "Jenkinsfile" -and $_.Name -ne "README.md" } | Select-Object -ExpandProperty FullName'
                        """,
                        returnStdout: true
                    ).trim()

                    if (invalidFiles) {
                        echo "❌ WHY FAILED: These files are NOT allowed:"
                        echo "${invalidFiles}"
                        error "❌ Invalid files found!"
                    } else {
                        echo "✅ All files are valid!"
                    }
                }
            }
        }
    }
}
