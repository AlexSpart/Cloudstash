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
        stage('Run VT') {
            environment {
                DEPLOY_FILE = 'cloudstash.csar'
                VT_DOCKER_NAME = 'RadonVT'
                VT_DOCKER_IMAGE = 'marklawimperial/verification-tool'
                VT_FILES_PATH = '{"path":"/tmp/main.cdl"}'
            }
            steps {
                // Pull the latest image of VT
                sh 'docker pull $VT_DOCKER_IMAGE'
                // Unzip the csar
                sh 'unzip $DEPLOY_FILE'
                // Move relevant files to temp folder. 
                sh 'mkdir -p tmp/radon-vt && cp -r _definitions tmp/radon-vt/_definitions && cp radon-vt/main.cdl tmp/radon-vt'
                // Run Verification Tool as Docker image and open port 5000 
                sh 'docker run --name $VT_DOCKER_NAME --rm -d -p 5000:5000 -v $PWD/tmp/radon-vt:/tmp $VT_DOCKER_IMAGE'
                // Wait some sec for the container to spin up.
                sh 'sleep 5'
                // Verify the model with the main.cdl restrictions. - Detect inconsistencies.
                sh 'curl -X POST -H "Content-type: application/json" http://localhost:5000/solve/ -d $VT_FILES_PATH'
                // Correct the model to comply with the main.cdl restrictions. - Propose correction of inconsistencies.
                sh 'curl -X POST -H "Content-type: application/json" http://localhost:5000/correct/ -d $VT_FILES_PATH'
                // Stop the container
                sh 'docker stop $VT_DOCKER_NAME'
            }
        }
        stage('Opera deploy') {
            environment {
                DEPLOY_FILE = 'ThumbnailGeneration.csar'
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