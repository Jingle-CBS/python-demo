pipeline {
    agent any

    environment {
        APP_NAME = 'python-demo'
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install flake8
                    flake8 app.py test_app.py --max-line-length=120
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pytest test_app.py -v --junitxml=test-results.xml
                '''
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'tar -czf ${APP_NAME}-${BUILD_TAG}.tar.gz app.py requirements.txt'
                archiveArtifacts artifacts: "${APP_NAME}-${BUILD_TAG}.tar.gz", fingerprint: true
            }
        }
    }

    post {
        success {
            echo '✅ 构建成功：代码检查通过 + 测试通过 + 制品已归档'
        }
        failure {
            echo '❌ 构建失败，请检查 Console Output'
        }
        always {
            cleanWs()
        }
    }
}
