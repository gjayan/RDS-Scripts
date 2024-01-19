pipeline {
    agent {
        docker {
          image 'liquibase/liquibase:4.4.2'
        }
    }

    parameters {
      choice choices: ['Update', 'Rollback'], description: 'Please select the operation to be performed', name: 'operationParam'
      choice choices: ['basic', 'state'], description: 'Please select the schema', name: 'schemaParam'
    }

    environment {
      endpoint = "database-1.cpogtkusv37n.ap-south-1.rds.amazonaws.com:3306"
      RDS_CREDS=credentials('RDS-CREDS')
    }

    stages {
      stage('Liquibase version'){
        steps{
          sh 'liquibase --version'
        }
      }
      stage('SCM stage'){
        steps{
            git branch: 'main', url: 'https://github.com/gjayan/RDS-Scripts.git'
        }
      }
      stage('Validating SQL'){
        steps{
          echo'*************LIQUIBASE VALIDATION*********************'
          sh 'liquibase validate --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
        }
      }
      stage('Status'){
        steps{
          script{
            echo '****************LIQUIBASE STATUS************************'
            sh 'liquibase status --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
            echo '${operationParam} will be performed on ${schemaParam}'
            env.Proceed = input message: 'Do you wish to proceed', id: "DB-${BUILD_NUMBER}", ok: 'Proceed',
            parameters: [choice(name: 'proceedParam', choices: ['Yes', 'No'], description: 'Please choose to proceed')]
          }
        }
      }
      stage('Update'){
        when {
          allOf {
            environment name: 'Proceed', value: 'Yes'
            expression {
              params.operationParam == 'Update'
            }
          }
        }
        steps{
          script {
            echo "************LIQUIBASE UPDATE*************************"
            sh 'liquibase update --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
            echo 'The ${schemaParam} has been Updated'
          }
        }
      }      
      stage('Rollback'){
        when {
          allOf {
            environment name:'Proceed', value: 'Yes'
            expression { params.operationParam = 'Rollback'}
          }
        }
        steps{
          script{
            echo '*************LIQUIBASE ROLLBACK***************************'
            sh 'liquibase rollback-count --count=1 --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
            echo 'The ${schemaParam} has been Rolled back'
          }
        }
      }
    }

    post{
      always{
        echo 'Cleaning up the workspace'
        sh 'rm -rf *'
        echo 'Workspace has been cleared'
      }
    }
}
