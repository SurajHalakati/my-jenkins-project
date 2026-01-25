pipeline {
    agent any

    parameters {
        string(
            name: 'BRANCH_NAME',
            defaultValue: '',
            description: 'Enter branch name manually'
        )
    }

    stages {

        stage('Check Branch Input') {
            steps {
                script {
                    if (!params.BRANCH_NAME?.trim()) {
                        error "❌ Branch name is empty! Please enter a branch name and run again."
                    }
                    echo "✅ Branch entered: ${params.BRANCH_NAME}"
                }
            }
        }

        stage('Validate Branch & Checkout') {
            steps {
                script {
                    def repoUrl = "https://github.com/Soumyakc-161/test-repo-jenkiness.git"

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

