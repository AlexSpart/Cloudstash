pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
    }
    stages {
        stage('Install dependacies') {
            steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'echo install any dependancies..'
                }
            }
        }

        stage('Opera Deploy') {
            environment {
                DEPLOY_FILE = 'ThumbnailGeneration.csar'
                OPERA_DOCKER_NAME = 'prqContOpera'
            }
            steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {  
                    // install the necessary dependencies as pip packages
                    // unwrap the csar and deploy the file.
                    sh '''
                        pip3 install ansible opera --user
                        PATH="$(python3 -m site --user-base)/bin:${PATH}"
                        opera init $DEPLOY_FILE
                        opera deploy
                    '''
                }
    
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}