pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
    }
    stages {
        stage('Run CTT') {
            environment {
                CTT_SERVER_URL = 'http://localhost:18080/RadonCTT)'
                CTT_CONFIG = ''
            }
            steps {
                sh 'docker pull radonconsortium/radon-ctt:latest'
                sh 'docker run -t -i --name RadonCTT -p 18080:18080 -v /var/run/docker.sock:/var/run/docker.sock radonconsortium/radon-ctt:latest'
                sh '''
                    python -m venv .venv && . .venv/bin/activate 
                    pip install -r requirements.txt
                    pip list
                    '''
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}   