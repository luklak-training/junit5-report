pipeline {
    agent any

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
                    def htmlFilePath = 'target/site/surefire-report.html'  // Đường dẫn tới file HTML của bạn
                    def htmlContent = readFile(htmlFilePath)

                    // Parse HTML để lấy thông tin từ phần Summary
                    def summaryData = parseHtmlForSummary(htmlContent)

                    // Tạo nội dung tin nhắn Telegram
                    def message = """
                        📋 *Surefire Test Summary*\n\n
                        *Tests*: ${summaryData.tests}\n
                        *Errors*: ${summaryData.errors}\n
                        *Failures*: ${summaryData.failures}\n
                        *Skipped*: ${summaryData.skipped}\n
                        *Success Rate*: ${summaryData.successRate}\n
                        *Time*: ${summaryData.time}\n
                    """

                    // Gửi tin nhắn đến Telegram
                    sendToTelegram(message)
//                             sh """
//                                 curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \\
//                                     -d chat_id=${TELEGRAM_CHAT_ID} \\
//                                     -d text="${message}"
//                             """
                        } else {
                            echo "✅ Tests passed successfully!"
                        }
                    }
                }
    }
}

def parseHtmlForSummary(htmlContent) {
    def parser = new XmlParser()
    def root = parser.parseText(htmlContent)
    def summaryTable = null

    root.depthFirst().each { node ->
        if (node.name() == 'table') {
            def thTexts = []
            node.tr[0].th.each { th ->
                thTexts.add(th.text())
            }
            if (thTexts.contains('Tests')) {
                summaryTable = node
            }
        }
    }

    if (summaryTable == null) {
        error("Summary table not found in HTML!")
    }

    def headers = []
    summaryTable.tr[0].th.each { th ->
        headers.add(th.text())
    }

    def values = []
    summaryTable.tr[1].td.each { td ->
        values.add(td.text())
    }

    def summaryData = [
        tests      : values[0],
        errors     : values[1],
        failures   : values[2],
        skipped    : values[3],
        successRate: values[4],
        time       : values[5]
    ]

    return summaryData
}


    def sendToTelegram(message) {
        // Gửi thông điệp tới Telegram qua API Bot
        def url = "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage"
        def payload = [
            chat_id: TELEGRAM_CHAT_ID,
            text: message,
            parse_mode: 'Markdown'
        ]

        // Sử dụng HTTP Request để gửi
        httpRequest(
            url: url,
            httpMode: 'POST',
            contentType: 'APPLICATION_JSON',
            requestBody: groovy.json.JsonOutput.toJson(payload)
        )
    }