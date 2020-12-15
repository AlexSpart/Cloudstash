pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
    }
    stages {
        stage('Run CTT') {
            environment {
                CTT_SERVER_URL = 'http://localhost:18080/RadonCTT'
                CTT_CONFIG = '$PWD/radon-ctt-cli-testing/ctt_config.yaml'
            }
            steps {
                sh 'docker stop RadonCTT'
                sh 'docker pull radonconsortium/radon-ctt:latest'
                sh 'docker run -d --rm --name RadonCTT -p 18080:18080 -v /var/run/docker.sock:/var/run/docker.sock radonconsortium/radon-ctt:latest'
                sh '''
                    git clone https://github.com/radon-h2020/radon-ctt-cli.git
                    ls
                    python3 -m venv .venv 
                    . .venv/bin/activate 
                    pip install -r radon-ctt-cli/requirements.txt
                    pip list
                    ls
                    python radon-ctt-cli/ctt_cli.py -u $CTT_SERVER_URL -c $CTT_CONFIG
                    '''
                sh 'docker stop RadonCTT'
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}   