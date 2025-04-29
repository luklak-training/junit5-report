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
                def testSummary = parseTestSummary()

                def message = """
🚨 Jenkins Test Result:
🛠️ Job: ${env.JOB_NAME}
🔢 Build: #${env.BUILD_NUMBER}
✅ Passed: ${testSummary.passed}
❌ Failed: ${testSummary.failed}
⚠️ Skipped: ${testSummary.skipped}
🧪 Total: ${testSummary.total}
⏱️ Time: ${testSummary.time} seconds
📄 Report: ${env.BUILD_URL}target/site/surefire-report.html
"""
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

def parseTestSummary() {
    def summary = [
        total: 0,
        failed: 0,
        skipped: 0,
        passed: 0,
        time: 0.0
    ]

    def testResultFiles = findFiles(glob: 'target/surefire-reports/TEST-*.xml')
    for (file in testResultFiles) {
        def content = readFile(file.path)
        def testSuite = new XmlSlurper().parseText(content)

        int tests = testSuite.@tests.toInteger()
        int failures = testSuite.@failures.toInteger()
        int errors = testSuite.@errors.toInteger()
        int skipped = (testSuite.@skipped.text() ?: "0").toInteger() // Nếu rỗng thì 0
        double time = (testSuite.@time.text() ?: "0").toDouble()

        summary.total += tests
        summary.failed += failures + errors
        summary.skipped += skipped
        summary.time += time
    }
    summary.passed = summary.total - summary.failed - summary.skipped
    return summary
}