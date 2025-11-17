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
                    echo Checking if permission set is already assigned...

                    sf data query -q "SELECT Id FROM PermissionSetAssignment WHERE Assignee.Username='anirudhabhi8008230@agentforce.com' AND PermissionSet.Name='least_priv_vehicle'" -o %ORG_ALIAS% > check_ps.txt

                    findstr /C:"records\": []" check_ps.txt >nul
                    if %errorlevel%==0 (
                        echo Permission not assigned. Assigning now...
                        sf force:user:permset:assign -n least_priv_vehicle -u %ORG_ALIAS%
                    ) else (
                        echo Permission already assigned. Skipping...
                    )
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
