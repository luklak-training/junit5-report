pipeline {
    agent any

triggers {
        cron('* * * * *')
    }

    environment {
                    TELEGRAM_TOKEN = credentials('telegram-bot-token')
                    TELEGRAM_CHAT_ID = credentials('telegram-chat-id')
                    TEST_FAILED = false
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                script {
                def testFailed = false
                                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                        sh 'mvn clean test'
                                    }
                                    if (currentBuild.currentResult == 'FAILURE' || currentBuild.currentResult == 'UNSTABLE') {
                                        echo "✅ Tests fail!"
                                       testFailed = true
                                    }
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
                always {
                    script {
                        if (currentBuild.result == 'FAILURE' || currentBuild.result == 'UNSTABLE') {
                    def tests = sh(script: "grep -oP '<td[^>]*>\\K[0-9]+(?=</td>)' target/site/surefire-report.html | sed -n 1p", returnStdout: true).trim()
                    def errors = sh(script: "grep -oP '<td[^>]*>\\K[0-9]+(?=</td>)' target/site/surefire-report.html | sed -n 2p", returnStdout: true).trim()
                    def failures = sh(script: "grep -oP '<td[^>]*>\\K[0-9]+(?=</td>)' target/site/surefire-report.html | sed -n 3p", returnStdout: true).trim()
                    def skipped = sh(script: "grep -oP '<td[^>]*>\\K[0-9]+(?=</td>)' target/site/surefire-report.html | sed -n 4p", returnStdout: true).trim()
                    def successRate = sh(script: "grep -oP '(?<=<td>)[0-9.]+%(?=</td>)' target/site/surefire-report.html | head -n 1", returnStdout: true).trim()
                    def time = sh(script: "grep -oP '(?<=<td>)[0-9.]+(?= s</td>)' target/site/surefire-report.html | head -n 1", returnStdout: true).trim()

                    def message = """
                        📋 *Surefire Test Summary*\n\n
                        *Tests*: ${tests}\n
                        *Errors*: ${errors}\n
                        *Failures*: ${failures}\n
                        *Skipped*: ${skipped}\n
                        *Success Rate*: ${successRate}\n
                        *Time*: ${time} seconds\n
                    """
                    // Gửi tin nhắn đến Telegram
//                     sendToTelegram(message)
                            sh """
                                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \\
                                    -d chat_id=${TELEGRAM_CHAT_ID} \\
                                    -d text="${message}"
                            """
                        } else {
                            echo "✅ Tests passed successfully!"
                        }
                    }
                }
    }
}