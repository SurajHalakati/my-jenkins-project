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
                    } catch (err) {
                        error "❌ Branch validation failed. Branch not found: ${params.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Validate Files') {
            steps {
                script {
                    echo "✅ Validating allowed files..."

                    try {
                        def invalidFiles = sh(
                            script: """
                            find . -type f \
                            ! -name '*.json' \
                            ! -name 'Jenkinsfile' \
                            ! -path './.git/*'
                            """,
                            returnStdout: true
                        ).trim()

                        if (invalidFiles) {
                            error "❌ Invalid files detected:\n${invalidFiles}"
                        }

                        echo "✅ File validation passed"
                    } catch (err) {
                        echo "⚠ Validate Files failed — continuing for learning"
                    }
                }
            }
        }

        stage('ARM Validate - Resource Group') {
            steps {
                echo "✅ Validating Resource Group ARM template..."

                sh """
                az deployment group validate \
                  --resource-group rg-validation \
                  --template-file azure-adf-e2e/arm-templates/resource-group/rg.json
                """
            }
        }

        stage('ARM Validate - Data Factory') {
            steps {
                echo "✅ Validating Data Factory ARM template..."

                sh """
                az deployment group validate \
                  --resource-group rg-validation \
                  --template-file azure-adf-e2e/arm-templates/data-factory/ARMTemplateForFactory.json \
                  --parameters azure-adf-e2e/arm-templates/data-factory/ARMTemplateParametersForFactory.json
                """
            }
        }

        stage('ARM What-If - Data Factory') {
            steps {
                echo "🔍 Previewing Azure changes (What-If)..."

                sh """
                az deployment group what-if \
                  --resource-group rg-validation \
                  --template-file azure-adf-e2e/arm-templates/data-factory/ARMTemplateForFactory.json \
                  --parameters azure-adf-e2e/arm-templates/data-factory/ARMTemplateParametersForFactory.json
                """
            }
        }

        stage('Publish ARM Artifact') {
            steps {
                echo "📦 Publishing validated ARM templates as artifact..."

                archiveArtifacts artifacts: 'azure-adf-e2e/arm-templates/**/*.json',
                                 fingerprint: true
            }
        }

        stage('Success') {
            steps {
                echo "✅ CI Pipeline completed"
            }
        }
    }

    post {
        failure {
            echo "❌ PIPELINE FAILED"
            echo "👉 Check the RED stage above"
            echo "👉 Open Console Output"
            echo "👉 Look for 'ERROR' or 'az deployment' messages"
        }

        success {
            echo "🎉 PIPELINE SUCCESS"
        }

        always {
            echo "ℹ Pipeline execution finished"
        }
    }
}
