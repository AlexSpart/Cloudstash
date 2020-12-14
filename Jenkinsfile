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
        stage('Run DPT') {
            environment {
                DEPLOY_FILE = 'cloudstash.csar'
                DPT_DOCKER_IMAGE = 'radonconsortium/radon-dp'
                GITHUB_REPO = 'AlexSpart/Cloudstash'
            }
            steps {
                // Pull radonconsortium/radon-dp:latest image from Dockerhub
                sh 'docker pull $DPT_DOCKER_IMAGE'
                // Create temporary
                sh 'mkdir -p tmp/radon-dp-volume'
                // Download a suitable model 
                sh 'docker run -v tmp/radon-dp-volume:/app $DPT_DOCKER_IMAGE radon-defect-predictor download-model tosca github $GITHUB_REPO'
                // Move CSAR into tmp library. This is colocated with the fetched radondp_model.joblib
                sh 'cp $DEPLOY_FILE tmp/radon-dp-volume'
                // Run predict
                sh 'docker run -v tmp/radon-dp-volume:/app $DPT_DOCKER_IMAGE radon-defect-predictor predict tosca $DEPLOY_FILE'
                // Results are available at:
                sh 'cat tmp/radon-dp-volume/radondp_predictions.json'
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}   