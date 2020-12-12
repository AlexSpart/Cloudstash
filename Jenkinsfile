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
                // Unzip the csar
                sh 'unzip -o ${DEPLOY_FILE}'
                // Move relevant files to temp folder. 
                sh 'mkdir -p tmp/radon-vt && cp -r _definitions tmp/radon-vt/_definitions && cp radon-vt/main.cdl tmp/radon-vt'
                // Run Verification Tool as Docker image and open port 5000 
                sh 'docker run --name "${VT_DOCKER_NAME}" --rm -d -p 5000:5000 -v /tmp/radon-vt:/tmp/radon/container marklawimperial/verification-tool'
                // Verify the model with the main.cdl restrictions. 
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