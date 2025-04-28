pipeline {
    agent any

    environment {
                    TELEGRAM_TOKEN = credentials('telegram-bot-token')
                    TELEGRAM_CHAT_ID = credentials('telegram-chat-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('Generate Report') {
            steps {
                sh 'mvn surefire-report:report'
            }
        }
        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'target/site',
                    reportFiles: 'surefire-report.html',
                    reportName: 'JUnit5 Test Report',
                    keepAll: true
                ])
            }
        }
    }

    post {
        failure {
            script {
                def message = """
🚨 Jenkins Build FAILED!
🛠️ Job: ${env.JOB_NAME}
🔢 Build Number: #${env.BUILD_NUMBER}
🔗 Link: ${env.BUILD_URL}
"""
                sh """
                    curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \\
                        -d chat_id=${TELEGRAM_CHAT_ID} \\
                        -d text="${message}"
                """
            }
        }
    }
}
