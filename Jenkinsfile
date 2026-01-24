pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL',
               defaultValue: 'https://github.com/SurajHalakati/my-jenkins-project.git',
               description: 'Repo URL')

        string(name: 'BRANCH_NAME',
               defaultValue: 'main',
               description: 'Enter Branch Name')

        string(name: 'RESOURCE_GROUP',
               defaultValue: 'cft-rg',
               description: 'Resource Group Name')

        choice(name: 'LOCATION',
               choices: ['southindia', 'eastus', 'centralindia'],
               description: 'Azure Location')

        choice(name: 'ACTION',
               choices: ['VALIDATE', 'WHAT_IF', 'DEPLOY'],
               description: 'Select Action')
    }

    stages {

        stage("Check Branch Name") {
            steps {
                script {
                    echo "Branch Selected: ${params.BRANCH_NAME}"

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

        // ✅ IMPORTANT DEBUG STAGE (this will show what Jenkins downloaded)
        stage("List Files") {
            steps {
                bat "echo ===== ROOT FILES ====="
                bat "dir"

                bat "echo ===== ARM TEMPLATES FOLDER ====="
                bat "dir arm-templates"

                bat "echo ===== STORAGE FOLDER ====="
                bat "dir arm-templates\\storage"

                bat "echo ===== ADF FOLDER ====="
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
                        echo "Checking file: ${f}"
                        if (!fileExists(f)) {
                            error("Missing file: ${f}")
                        }
                        if (!f.toLowerCase().endsWith(".json")) {
                            error("Only .json allowed. Wrong file: ${f}")
                        }
                    }

                    echo "All required files found"
                }
            }
        }

        stage("JSON Syntax Check") {
            steps {
                bat """
                powershell -Command "Get-ChildItem -Recurse -Filter *.json | ForEach-Object { Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null }"
                """
                echo "JSON syntax OK"
            }
        }

        stage("Azure Login") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-sp',
                    usernameVariable: 'AZ_CLIENT_ID',
                    passwordVariable: 'AZ_CLIENT_SECRET'
                )]) {
                    bat """
                    az login --service-principal ^
                      -u %AZ_CLIENT_ID% ^
                      -p %AZ_CLIENT_SECRET% ^
                      --tenant YOUR_TENANT_ID
                    """
                }
            }
        }

        stage("Create Resource Group") {
            when {
                expression { return params.ACTION == 'DEPLOY' }
            }
            steps {
                script {
                    def rgExists = bat(
                        script: "az group exists --name ${params.RESOURCE_GROUP}",
                        returnStdout: true
                    ).trim()

                    if (rgExists == "true") {
                        error("Resource Group already exists: ${params.RESOURCE_GROUP}")
                    }

                    bat """
                    az group create --name ${params.RESOURCE_GROUP} --location ${params.LOCATION}
                    """
                }
            }
        }

        stage("ARM Validate") {
            when {
                expression { return params.ACTION == 'VALIDATE' }
            }
            steps {
                bat """
                az deployment group validate ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file arm-templates/storage/storage.json ^
                  --parameters @arm-templates/storage/storage.parameters.json
                """

                bat """
                az deployment group validate ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file arm-templates/adf/ArmTemplate_master.json ^
                  --parameters @arm-templates/adf/ArmTemplateParameters_master.json
                """
            }
        }

        stage("ARM What-If") {
            when {
                expression { return params.ACTION == 'WHAT_IF' }
            }
            steps {
                bat """
                az deployment group what-if ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file arm-templates/storage/storage.json ^
                  --parameters @arm-templates/storage/storage.parameters.json
                """

                bat """
                az deployment group what-if ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file arm-templates/adf/ArmTemplate_master.json ^
                  --parameters @arm-templates/adf/ArmTemplateParameters_master.json
                """
            }
        }

        stage("Deploy Storage + ADF") {
            when {
                expression { return params.ACTION == 'DEPLOY' }
            }
            steps {
                bat """
                az deployment group create ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --mode Complete ^
                  --template-file arm-templates/storage/storage.json ^
                  --parameters @arm-templates/storage/storage.parameters.json
                """

                bat """
                az deployment group create ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --mode Complete ^
                  --template-file arm-templates/adf/ArmTemplate_master.json ^
                  --parameters @arm-templates/adf/ArmTemplateParameters_master.json
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline success"
        }
        failure {
            echo "Pipeline failed. Check console output"
        }
    }
}
