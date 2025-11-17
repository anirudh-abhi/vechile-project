pipeline {
    agent any

    environment {
        SF_AUTH = credentials('SF_AUTH_FILE')
        ORG_ALIAS = "mySandbox"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore Salesforce Auth') {
            steps {
                bat '''
                    echo Restoring authfile...
                    copy "%SF_AUTH%" authfile.json

                    echo Authenticating to Salesforce...
                    sf org login sfdx-url --sfdx-url-file authfile.json --alias %ORG_ALIAS%
                '''
            }
        }

        stage('Deploy Metadata') {
            steps {
                bat '''
                    echo Deploying metadata...
                    sf project deploy start --target-org %ORG_ALIAS% --source-dir force-app/main/default --wait 20
                '''
            }
        }

        stage('Assign Permission Set') {
            steps {
                bat '''
                    echo Assigning permission set...
                    sf force:user:permset:assign -n least_priv_vehicle -u %ORG_ALIAS%
                '''
            }
        }

        stage('Load Data') {
            when {
                expression { fileExists('vehicle_data.csv') }
            }
            steps {
                bat '''
                    echo Loading CSV data into Salesforce...
                    sf data bulk insert -s Info_Vehicle__c -f vehicle_data.csv -o %ORG_ALIAS% --wait 600
                '''
            }
        }

        stage('Verify Records') {
            steps {
                bat '''
                    echo Checking record count...
                    sf data query -q "SELECT COUNT() FROM Info_Vehicle__c" -o %ORG_ALIAS%
                '''
            }
        }
    }

    post {
        success {
            echo "✔ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
