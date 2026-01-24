pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL',
               defaultValue: 'https://github.com/Soumyakc-161/test-repo-jenkiness.git',
               description: 'GitHub Repo URL')

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

        string(name: 'STORAGE_TEMPLATE',
               defaultValue: 'azure-adf-e2e/arm-template/storage-account/storage.json',
               description: 'Storage ARM Template (.json)')

        string(name: 'STORAGE_PARAMS',
               defaultValue: 'azure-adf-e2e/arm-template/storage-account/storage.parameters.json',
               description: 'Storage Parameters (.json)')

        string(name: 'ADF_TEMPLATE',
               defaultValue: 'azure-adf-e2e/arm-template/data-factory/linkedTemplates/ArmTemplate_master.json',
               description: 'ADF ARM Template (.json)')

        string(name: 'ADF_PARAMS',
               defaultValue: 'azure-adf-e2e/arm-template/data-factory/linkedTemplates/ArmTemplateParameters_master.json',
               description: 'ADF Parameters (.json)')
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

        stage("Check Required Files") {
            steps {
                script {
                    def files = [
                        params.STORAGE_TEMPLATE,
                        params.STORAGE_PARAMS,
                        params.ADF_TEMPLATE,
                        params.ADF_PARAMS
                    ]

                    for (f in files) {
                        if (!fileExists(f)) {
                            error("Missing file: ${f}")
                        }
                        if (!f.toLowerCase().endsWith(".json")) {
                            error("Only .json files allowed. Wrong file: ${f}")
                        }
                    }

                    echo "All required files found"
                }
            }
        }

        stage("JSON Syntax Check") {
            steps {
                script {
                    def jsonFiles = [
                        params.STORAGE_TEMPLATE,
                        params.STORAGE_PARAMS,
                        params.ADF_TEMPLATE,
                        params.ADF_PARAMS
                    ]

                    for (f in jsonFiles) {
                        echo "Checking JSON: ${f}"
                        bat """
                        powershell -Command "Get-Content '${f}' -Raw | ConvertFrom-Json | Out-Null"
                        """
                    }

                    echo "JSON syntax is valid"
                }
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
                        error("Resource Group '${params.RESOURCE_GROUP}' already exists. Pipeline failed.")
                    }

                    bat """
                    az group create ^
                      --name ${params.RESOURCE_GROUP} ^
                      --location ${params.LOCATION}
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
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                bat """
                az deployment group validate ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.ADF_TEMPLATE} ^
                  --parameters @${params.ADF_PARAMS}
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
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                bat """
                az deployment group what-if ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.ADF_TEMPLATE} ^
                  --parameters @${params.ADF_PARAMS}
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
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                bat """
                az deployment group create ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --mode Complete ^
                  --template-file ${params.ADF_TEMPLATE} ^
                  --parameters @${params.ADF_PARAMS}
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
