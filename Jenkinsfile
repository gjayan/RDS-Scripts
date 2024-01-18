pipeline {
    agent {
        docker {
          image 'liquibase/liquibase:4.4.2'
        }
    }

    parameters {
      choice choices: ['update', 'rollback'], description: 'Please select the operation to be performed', name: 'operationParam'
      choice choices: ['basic', 'state'], description: 'Please select the schema', name: 'schemaParam'
    }

    environment {
      endpoint = "database-1.cpogtkusv37n.ap-south-1.rds.amazonaws.com:3306"
      RDS_CREDS=credentials('RDS-CREDS')
    }

    stages {
      stage('Clean Up'){
        steps{
           // deleteDir()
            sh 'pwd'
            sh 'ls'
        }
      }
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
            env.Proceed = input message: 'Please select to proceed', id: "Update-${BUILD_NUMBER}", ok: 'Proceed',
            parameters: [choice(name: 'proceedParam', choices: ['Yes', 'No'], description: 'Please choose to proceed')]
          }
        }
      }
      stage('Update'){
        when{
          expression {
            params.operationParam = 'update'
          }
        }
        steps{
          script {
            echo "************LIQUIBASE UPDATE*************************"
            sh 'liquibase update --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
            echo 'The db has been updated'
          }
        }
      }
      stage('Rollback'){
        when{
          expression{
            params.operationParam = 'rollback'
          }
        }
        steps{
          script{
            echo '*************LIQUIBASE ROLLBACK***************************'
            sh 'liquibase rollback-count --count=1 --url="jdbc:mariadb://${endpoint}/${schemaParam}" --changeLogFile=./${schemaParam}/changelogFile.xml --username=$RDS_CREDS_USR --password=$RDS_CREDS_PSW'
            echo 'The db commit has been rollback'
          }
        }
      }
    }

    post{
      always{
        echo 'Cleaning up the workspace'
        sh ' ls'
        sh 'rm -rf *'
        sh 'ls'
        echo 'Workspace has been cleared'
      }
    }
}
