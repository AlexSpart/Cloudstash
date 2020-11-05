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
        stage('Radon VT') {
            environment {
                DEPLOY_FILE = 'cloudstash.csar'
                VT_DOCKER_NAME = 'RadonVT'
                VT_FILES_PATH = '{"path":"/tmp/radon/container/main.cdl"}'
            }
            steps {
                sh 'echo Start VT container and perform verify test...'
                sh 'unzip -o ${DEPLOY_FILE}'
                sh 'mkdir -p $PWD/tmp/radon && cp -r _definitions $PWD/tmp/radon/_definitions && cp /radon-vt/main.cdl $PWD/tmp/radon'
                sh 'docker run --name "${VT_DOCKER_NAME}" --rm -d -p 5000:5000 -v $PWD/tmp/radon:/tmp/radon/container marklawimperial/verification-tool'
                sh 'sleep 5'
                sh 'docker exec ${VT_DOCKER_NAME} sh -c "cd /tmp/radon/container && pwd && ls -al && cat main.cdl"'
                sh 'curl -X POST -H "Content-type: application/json" http://localhost:5000/solve/ -d ${VT_FILES_PATH}'
                sh 'docker stop ${VT_DOCKER_NAME}'
            }
        }
        stage('Run DPT') {
            environment {
                DEPLOY_FILE = 'cloudstash.csar'
                DPT_DOCKER_NAME = 'RadonDPT'
            }
            steps {
                sh 'echo Start DPT container and perform test...'
                sh 'mkdir -p $PWD/radon-dpt && cp -r $DEPLOY_FILE $PWD/radon-dpt'
                sh 'git clone https://github.com/radon-h2020/radon-defect-prediction-cli.git'
                sh 'cd radon-defect-prediction-cli && pip3 install -r requirements.txt --user && pip3 install . --user'
                sh 'PATH="$(python3 -m site --user-base)/bin:${PATH}" && radon-defect-predictor download-model tosca github AlexSpart/radon-csars && radon-defect-predictor predict tosca $DEPLOY_FILE && python --version '
                sh 'cat radondp_predictions.json'
            }
        }
        stage('Opera Deploy') {
            environment {
                DEPLOY_FILE = 'cloudstash.csar'
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