pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/SurajHalakati/my-jenkins-project.git', description: 'Repo URL')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Enter Branch Name')
    }

    stages {

        stage("Check Branch Name") {
            steps {
                script {
                    def out = bat(
                        script: "git ls-remote --heads ${params.REPO_URL} ${params.BRANCH_NAME}",
                        returnStdout: true
                    ).trim()

                    if (!out || out.length() == 0) {
                        error("Branch '${params.BRANCH_NAME}' not found. Pipeline failed.")
                    }
                    echo "Branch found: ${params.BRANCH_NAME}"
                }
            }
        }

        stage("Checkout Repo") {
            steps {
                git branch: "${params.BRANCH_NAME}", url: "${params.REPO_URL}"
                echo "Checkout completed"
            }
        }

        stage("List Files") {
            steps {
                bat "dir"
                bat "dir arm-templates"
                bat "dir arm-templates\\storage"
                bat "dir arm-templates\\adf"
            }
        }

        stage("Check Required Files") {
            steps {
                script {
                    def files = [
                        "arm-templates/storage/storage.json",
                        "arm-templates/storage/storage.parameters.json",
                        "arm-templates/adf/ArmTemplate_master.json",
                        "arm-templates/adf/ArmTemplateParameters_master.json"
                    ]

                    for (f in files) {
                        if (!fileExists(f)) {
                            error("Missing file: ${f}")
                        }
                    }

                    echo "All required JSON files found"
                }
            }
        }

        stage("JSON Syntax Check") {
            steps {
                bat """
                powershell -Command "Get-ChildItem -Recurse -Filter *.json | ForEach-Object { Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null }"
                """
                echo "JSON syntax check passed"
            }
        }

        stage("Azure Validate") {
            steps {
                echo "SKIPPED: No Azure account / no Azure integration available."
            }
        }

        stage("Azure What-If") {
            steps {
                echo "SKIPPED: No Azure account / no Azure integration available."
            }
        }

        stage("Azure Deploy") {
            steps {
                echo "SKIPPED: No Azure account / no Azure integration available."
            }
        }
    }

    post {
        success {
            echo "Pipeline success (Git + JSON checks completed)"
        }
        failure {
            echo "Pipeline failed. Check console output"
        }
    }
}
