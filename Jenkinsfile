pipeline {
    agent any

    environment {
        SLACK_WEBHOOK_URL = credentials('slack-webhook')
        GMAIL_CREDENTIALS = credentials('gmail-smtp-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/tuusuario/tu-repo.git'
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Desplegando aplicación...'
            }
        }
    }

    post {
        success {
            slackSend (webhookUrl: "${SLACK_WEBHOOK_URL}", message: "✅ Build exitoso: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            emailext (
                subject: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "El build fue exitoso.",
                to: "andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com",
                from: "andres.vazquezleon01@gmail.com",
                smtpHost: "smtp.gmail.com",
                smtpPort: "587",
                mimeType: 'text/plain'
            )
        }
        failure {
            slackSend (webhookUrl: "${SLACK_WEBHOOK_URL}", message: "❌ Build fallido: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "El build falló. Verifica los logs.",
                to: "andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com",
                from: "andres.vazquezleon01@gmail.com",
                smtpHost: "smtp.gmail.com",
                smtpPort: "587",
                mimeType: 'text/plain'
            )
        }
    }
}
