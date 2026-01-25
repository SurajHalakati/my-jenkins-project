pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'feature/test_repo', description: 'Enter branch name')
    }

    stages {

        stage('Validate Branch & Checkout') {
            steps {
                script {
                    def repoUrl = "https://github.com/Soumyakc-161/test-repo-jenkiness.git"

                    echo "Branch entered: ${params.BRANCH_NAME}"

                    // Try to checkout the entered branch
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
    }
}
