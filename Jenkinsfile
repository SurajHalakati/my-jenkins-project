pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL',
               defaultValue: 'https://github.com/Soumyakc-161/test-repo-jenkiness.git',
               description: 'GitHub Repo URL')

        string(name: 'GIT_BRANCH',
               defaultValue: 'main',
               description: 'Branch name must exist, else pipeline will fail')

        string(name: 'RESOURCE_GROUP',
               defaultValue: 'cft-rg',
               description: 'Azure Resource Group to create')

        choice(name: 'LOCATION',
               choices: ['southindia', 'eastus', 'centralindia'],
               description: 'Azure region for RG')

        // ✅ ARM JSON template paths (update based on your repo)
        string(name: 'STORAGE_TEMPLATE',
               defaultValue: 'azure-adf-e2e/arm-template/storage-account/storage.json',
               description: 'Storage ARM template path (.json only)')

        string(name: 'STORAGE_PARAMS',
               defaultValue: 'azure-adf-e2e/arm-template/storage-account/storage.parameters.json',
               description: 'Storage parameters path (.json only)')

        string(name: 'ADF_TEMPLATE',
               defaultValue: 'azure-adf-e2e/arm-template/data-factory/linkedTemplates/ArmTemplate_master.json',
               description: 'ADF master template path (.json only)')

        string(name: 'ADF_PARAMS',
               defaultValue: 'azure-adf-e2e/arm-template/data-factory/linkedTemplates/ArmTemplateParameters_master.json',
               description: 'ADF parameters path (.json only)')

        // ✅ user chooses action
        choice(name: 'ACTION',
               choices: ['VALIDATE', 'WHAT_IF', 'DEPLOY'],
               description: 'Choose what you want pipeline to do')
    }

    stages {

        stage("0) Check Branch Name (Fail if wrong)") {
            steps {
                script {
                    echo "✅ Checking branch: ${params.GIT_BRANCH}"

                    def out = bat(
                        script: "git ls-remote --heads ${params.REPO_URL} ${params.GIT_BRANCH}",
                        returnStdout: true
                    ).trim()

                    if (out == null || out.length() == 0) {
                        error("❌ Branch '${params.GIT_BRANCH}' NOT found in repo. Pipeline failed.")
                    }

                    echo "✅ Branch exists: ${params.GIT_BRANCH}"
                }
            }
        }

        stage("1) Checkout Repo (Branch)") {
            steps {
                git branch: "${params.GIT_BRANCH}", url: "${params.REPO_URL}"
                echo "✅ Repo checkout completed | Branch = ${params.GIT_BRANCH}"
            }
        }

        stage("2) Check Required Files + Only .json (Fail if missing)") {
            steps {
                script {
                    def requiredFiles = [
                        params.STORAGE_TEMPLATE,
                        params.STORAGE_PARAMS,
                        params.ADF_TEMPLATE,
                        params.ADF_PARAMS
                    ]

                    for (f in requiredFiles) {
                        if (!fileExists(f)) {
                            error("❌ Missing file in repo: ${f}")
                        }
                        if (!f.toLowerCase().endsWith(".json")) {
                            error("❌ Only .json files allowed. Invalid file found: ${f}")
                        }
                    }

                    echo "✅ All required .json files present"
                }
            }
        }

        stage("3) JSON Syntax Check (Fail if syntax error)") {
            steps {
                script {
                    def jsonFiles = [
                        params.STORAGE_TEMPLATE,
                        params.STORAGE_PARAMS,
                        params.ADF_TEMPLATE,
                        params.ADF_PARAMS
                    ]

                    for (f in jsonFiles) {
                        echo "🔍 Checking JSON syntax: ${f}"
                        // Powershell will show the exact error line in console if JSON is invalid
                        bat """
                        powershell -Command "Get-Content '${f}' -Raw | ConvertFrom-Json | Out-Null"
                        """
                    }

                    echo "✅ JSON syntax check passed"
                }
            }
        }

        stage("4) Azure Login") {
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

        stage("5) Create RG (Fail if already exists)") {
            when {
                expression { return params.ACTION == 'DEPLOY' }
            }
            steps {
                script {
                    echo "✅ Checking Resource Group exists or not..."

                    def rgExists = bat(
                        script: "az group exists --name ${params.RESOURCE_GROUP}",
                        returnStdout: true
                    ).trim()

                    if (rgExists == "true") {
                        error("❌ Resource Group '${params.RESOURCE_GROUP}' already exists. Pipeline failed (as per requirement).")
                    }

                    echo "🚀 Creating Resource Group..."
                    bat """
                    az group create ^
                      --name ${params.RESOURCE_GROUP} ^
                      --location ${params.LOCATION}
                    """
                }
            }
        }

        stage("6) ARM Validate") {
            when {
                expression { return params.ACTION == 'VALIDATE' }
            }
            steps {
                echo "✅ Validating Storage template..."
                bat """
                az deployment group validate ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                echo "✅ Validating ADF template..."
                bat """
                az deployment group validate ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.ADF_TEMPLATE} ^
                  --parameters @${params.ADF_PARAMS}
                """
            }
        }

        stage("7) ARM What-If") {
            when {
                expression { return params.ACTION == 'WHAT_IF' }
            }
            steps {
                echo "✅ What-If Storage..."
                bat """
                az deployment group what-if ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                echo "✅ What-If ADF..."
                bat """
                az deployment group what-if ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --template-file ${params.ADF_TEMPLATE} ^
                  --parameters @${params.ADF_PARAMS}
                """
            }
        }

        stage("8) Deploy (Storage + ADF)") {
            when {
                expression { return params.ACTION == 'DEPLOY' }
            }
            steps {
                echo "🚀 Deploying Storage Account..."
                bat """
                az deployment group create ^
                  --resource-group ${params.RESOURCE_GROUP} ^
                  --mode Complete ^
                  --template-file ${params.STORAGE_TEMPLATE} ^
                  --parameters @${params.STORAGE_PARAMS}
                """

                echo "🚀 Deploying ADF (Linked Services + Pipelines)..."
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
            echo "✅ SUCCESS ✅ Pipeline completed for Branch: ${params.GIT_BRANCH}"
        }
        failure {
            echo "❌ FAILED ❌ Check console output (it will show exact error & file)."
        }
    }
}
