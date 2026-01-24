pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

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
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH_NAME}"]],
                    userRemoteConfigs: [[url: "${params.REPO_URL}"]]
                ])
                echo "Checkout completed"
            }
        }

        stage("Create Resource Group") {
            when {
