pipeline {
    agent any

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
                stage('gen report') {
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
}
