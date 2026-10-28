
pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub.'
            }
        }

        stage('Check Python') {
            steps {
                sh 'python3 --version'
                sh 'git --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Validate Python') {
            steps {
                sh '.venv/bin/python -m compileall -q app.py'
            }
        }

        stage('Test Flask Endpoints') {
            steps {
                sh '''
                    .venv/bin/python -c "
from app import app
client = app.test_client()
r1 = client.get('/')
assert r1.status_code == 200, r1.status_code
assert r1.json['application'] == 'Jenkins CI/CD Flask Backend'
r2 = client.get('/health')
assert r2.status_code == 200, r2.status_code
assert r2.json['status'] == 'healthy'
print('Flask endpoint tests passed.')
"
                '''
            }
        }
        stage('Deploy to EC2') {
            steps {
                sh 'sudo /usr/local/sbin/deploy-flask'
            }
        }
    }

    post {
        success {
            echo 'Flask CI pipeline completed successfully.'
        }
        failure {
            echo 'Flask CI pipeline failed. Review the console output.'
        }
    }
}
