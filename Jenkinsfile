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
                dir('backend') { // nếu pom.xml nằm trong thư mục backend
                    sh 'mvn clean test surefire-report:report'
                }
            }
        }
        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'backend/target/site',
                    reportFiles: 'surefire-report.html',
                    reportName: 'JUnit5 Test Report',
                    keepAll: true
                ])
            }
        }
    }
}
