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
                VT_FILES_PATH = '{"path":"/tmp/radon/container/main.cdl"}'
            }
            steps {
                sh 'unzip "${DEPLOY_FILE}"'
                sh 'ls -al'
                sh 'mkdir -p $PWD/tmp/radon && cp -r _definitions $PWD/tmp/radon/_definitions && cp $PWD/radon-vt/main.cdl $PWD/tmp/radon'
                sh 'docker run --name "${VT_DOCKER_NAME}" --rm -d -p 5000:5000 -v $PWD/tmp/radon:/tmp/radon/container marklawimperial/verification-tool'
                sh 'sleep 5'
                sh 'curl -X POST -H "Content-type: application/json" http://localhost:5000/solve/ -d ${VT_FILES_PATH}'
                sh 'docker stop ${VT_DOCKER_NAME}'
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}