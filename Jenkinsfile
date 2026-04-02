pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 10, unit: 'MINUTES') // Timeout after 10 minutes
        disableConcurrentBuilds() // Prevent concurrent builds
    }
    parameters {
       // booleanParams(name: "deploy", defaultValue: false, description: "Deploy to production?")
        choice(name: "ENVIRONMENT", choices: ["dev", "qa", "uat","pre-prod","prod"], description: "Select the deployment environment")
        string(name: "Version", description: "Application version to deploy")
        string(name: "jira-id, description: "JIRA ID for tracking deployment")
    }
    environment { 
        appVersion = '' // Can be set dynamically during the pipeline
        account_id = ''
        region = "us-east-1"
        project = "expense"
        environment = ''
        component = "backend"
    }

   
    stages {
        stage('setup environment') {
            steps {
                script {
                    environment = params.ENVIRONMENT
                    appVersion = params.Version
                    account_id = pipelineGlobals.getAccountId(environment)
                    // if (params.environment) {
                    //     environment = params.environment
                    // }
                    // if (params.app-Version) {
                    //     appVersion = params.app-Version
                    // }
                    // echo "Deployment Environment: ${environment}"
                    // echo "Application Version: ${appVersion}"
                }
            }
        }
        stage('Integration Tests') {
            when {
                expression {params.ENVIRONMENT == 'qa'}
            }
            steps{
                script {
                    echo "Running integration tests for ${environment} environment"
                    // Add your integration test commands here
                }

            }
        }
        stage('Check JIRA') {
             when {
                expression {params.ENVIRONMENT == 'prod'}
            }
            steps{
                script {
                    sh"""
                        echo "Checking JIRA for open issues related to ${component} in ${environment} environment"
                        echo "Check jira deployment window for any open issues before proceeding with deployment"
                        echo "fail pipeline if above two are not true"
                    """
                }

            }
        }
        stage('Deploy') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n ${project} --create-namespace -f values-${environment}.yaml .
                    """
                }
            }
         
        }
    }   
        // stage('Print Params'){
        //     steps{
        //         echo "Hello ${params.PERSON}"
        //         echo "Biography: ${params.BIOGRAPHY}"
        //         echo "Toggle: ${params.TOGGLE}"
        //         echo "Choice: ${params.CHOICE}"
        //         echo "Password: ${params.PASSWORD}"  
        //     }
        // }
        // stage('Approval') {
        //     input {
        //         message "Should we continue?.."
        //         ok "Yes, we should."
        //         submitter "alice,bob"
        //         parameters {
        //             string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        //         }
        //     }   
        //     steps {
        //         echo "Hello, ${PERSON}, nice to meet you."
        //     }
        // }
    
    post {
        always {
            echo 'This will always run'
            deleteDir() // Clean up workspace
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the pipeline has changed'
        }
    }
}