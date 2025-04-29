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
            // Đếm tổng tests
            def totalTests = sh(
                script: "grep -o 'tests=\"[0-9]\\+\"' target/surefire-reports/TEST-*.xml | sed 's/[^0-9]*//g' | awk '{sum+=\$1} END {print sum}'",
                returnStdout: true
            ).trim()

            def totalFailures = sh(
                script: "grep -o 'failures=\"[0-9]\\+\"' target/surefire-reports/TEST-*.xml | sed 's/[^0-9]*//g' | awk '{sum+=\$1} END {print sum}'",
                returnStdout: true
            ).trim()

            def totalErrors = sh(
                script: "grep -o 'errors=\"[0-9]\\+\"' target/surefire-reports/TEST-*.xml | sed 's/[^0-9]*//g' | awk '{sum+=\$1} END {print sum}'",
                returnStdout: true
            ).trim()

            def totalSkipped = sh(
                script: "grep -o 'skipped=\"[0-9]\\+\"' target/surefire-reports/TEST-*.xml | sed 's/[^0-9]*//g' | awk '{sum+=\$1} END {print sum}'",
                returnStdout: true
            ).trim()

            def totalTime = sh(
                script: "grep -o 'time=\"[0-9.]*\"' target/surefire-reports/TEST-*.xml | sed 's/[^0-9.]*//g' | awk '{sum+=\$1} END {print sum}'",
                returnStdout: true
            ).trim()

            echo "Tests=${totalTests}, Failures=${totalFailures}, Errors=${totalErrors}, Skipped=${totalSkipped}, Time=${totalTime}"

            // Tạo tin nhắn Telegram
            def message = """🧪 *JUnit5 Test Report* 🧪

*Total Tests*: ${totalTests}
*Failures*: ${totalFailures}
*Errors*: ${totalErrors}
*Skipped*: ${totalSkipped}
*Total Time*: ${totalTime}s

🔗 [View Report](${env.BUILD_URL}HTML_Report/)"""
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