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
    }

    stages {

        stage("Check Branch Name") {
            steps {
                script {
                    echo "Selected Branch: ${params.BRANCH_NAME}"

                    def out = bat(
                        script: "git ls-remote --heads ${params.REPO_URL} ${params.BRANCH_NAME}",
                        returnStdout: true
                    ).trim()

                    if (!out || out.length() == 0) {
                        error("Branch '${params.BRANCH_NAME}' not found. Pipeline failed.")
                    }

                    echo "Branch is correct ✅"
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
                echo "Repo checkout success ✅"
            }
        }

        stage("Build Success") {
            steps {
                echo "Pipeline passed successfully ✅"
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS"
        }
        failure {
            echo "❌ FAILED"
        }
    }
}
