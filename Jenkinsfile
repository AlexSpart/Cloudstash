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
                    sh 'echo Creating image opera-deploy...'
                    sh 'docker build --tag opera-deploy .'
                    sh 'echo Starting container prqCont...'
                    sh 'mkdir -p $PWD/tmp/radon && cp -r $DEPLOY_FILE $PWD/tmp/radon'
                    sh 'docker run --name ${OPERA_DOCKER_NAME} --rm -d -p 18080:18080 -v $PWD/tmp/radon:/tmp/radon/container -e "AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}" -e "AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}" -e "CTT_FAAS_ENABLED=1" opera-deploy'
                    sh 'sleep 5'
                    sh 'docker exec ${OPERA_DOCKER_NAME} sh -c "cd /tmp/radon/container && opera init $DEPLOY_FILE && opera deploy "'
                    sh 'docker stop $OPERA_DOCKER_NAME'
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