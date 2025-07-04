pipeline {
    agent { label 'AGENT-1'}
    environment { 
        PROJECT = 'expense'
        COMPONENT = 'frontend' 
        appVersion = ''
        ACC_ID = "879381244178"
        
    }

     options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    stages {
        stage('Read Version') {
            steps {
                script {
                  def packageJson = readJSON file: 'package.json'
                  appVersion = packageJson.version
                  echo "Version is: $appVersion"
                    
                }
               
            }
        }

        stage('Install Dependencies') {
            steps {
               script{ 
                 sh """
                    npm install
                 """
               }
            }
        }

        stage('Docker Build') {
            steps {
               script{
                withAWS(region: 'us-east-1', credentials: 'aws-creds-dev') {
                    sh """
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t  ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .

                    docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
                    """
                }
                 
               }
            }
        }

        stage('Trigger Deploy'){
            when { 
                 expression { params.deploy }
             }
             steps{
                build job: 'frontend-cd', parameters: [string(name: 'version', value: "${appVersion}")], wait: true
            }
        }

    }    
    
    post { 
    
        always { 
            echo 'I will always say hello again!'
            deleteDir()
        }
        failure { 
            echo 'This session runs when pipeline failure'
        }
        success { 
            echo 'This session runs when pipeline success'
        }
    }
}