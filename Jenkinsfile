pipeline {
    agent any

    environment {
        APP_NAME = 'python-demo'
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
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

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${APP_NAME}:${BUILD_TAG} .'
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh """
                    echo '🚀 Deploying ${APP_NAME}:${BUILD_TAG} to staging...'
                    docker stop ${APP_NAME}-staging || true
                    docker rm ${APP_NAME}-staging || true
                    docker run -d --name ${APP_NAME}-staging -p 8081:8080 ${APP_NAME}:${BUILD_TAG}
                    echo '✅ Staging is up at http://localhost:8081'
                """
            }
        }

        stage('Approval for Production') {
            steps {
                input message: 'Staging 已部署，确认发布到生产环境？', ok: '确认发布'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh """
                    echo '🚀 Deploying ${APP_NAME}:${BUILD_TAG} to PRODUCTION...'
                    docker stop ${APP_NAME}-prod || true
                    docker rm ${APP_NAME}-prod || true
                    docker run -d --name ${APP_NAME}-prod -p 8082:8080 ${APP_NAME}:${BUILD_TAG}
                    echo '✅ Production is up at http://localhost:8082'
                """
            }
        }
    }

    post {
        success {
            echo '🎉 全流程完成：代码检查 → 测试 → 打包 → 镜像 → 部署生产'
        }
        failure {
            echo '❌ 构建失败，请检查 Console Output'
        }
        always {
            cleanWs()
        }
    }
}
