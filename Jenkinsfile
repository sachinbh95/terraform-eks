pipeline {

    parameters {
       booleanParam(name: 'autoApprove', defaultValue: true, description: 'Automatically run apply after generating plan?')
       choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select the action to perform')
    }

    environment {
        // Set the default value for the environment variable
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_SESSION_TOKEN = credentials('AWS_SESSION_TOKEN')
    }

    agent any

    stages {
        stage('checkout') {
            steps {
                script {
                    // Checkout the code from the repository
                    git branch: 'main', url: 'https://github.com/sachinbh95/terraform-eks.git'
                }
            }
        }

        stage('terraform init') {
            steps {
                script {
                    // Initialize the terraform
                    sh 'terraform init'
                }
            }
        }

        stage('terraform plan') {
            steps {
                script {
                    // Plan the terraform
                    sh 'terraform plan -out tfplan'
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
            }
        }

        stage('Apply / Destroy') {
            steps {
                script {
                    if (params.action == 'apply') {
                        if (!params.autoApprove) {
                            def plan = readFile 'tfplan.txt'
                            input message: "Do you want to apply the plan?",
                            parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                        }

                        sh 'terraform ${action} -input=false tfplan'
                    } else if (params.action == 'destroy') {
                        sh 'terraform ${action} --auto-approve'
                    } else {
                        error "Invalid action selected. Please choose either 'apply' or 'destroy'."
                    }
                }
            }
        }
            
        stage('Trigger Cloud Deployment') {
            steps {
                script {
                    if (params.action == 'apply') {
                        build job: "cloud-deploy-eks", wait: true
                    }
                }
            }
        }

    }
}