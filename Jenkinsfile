pipeline {
    agent any

    environment {
        GMAIL_CREDENTIALS = credentials('gmail-smtp-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Andres-Vazquez-Leon/practica_ci_cd.git'
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
                // Aquí iría tu comando real de despliegue
            }
        }
    }

    post {
        success {
            // Notificación por Slack (configurado globalmente)
            slackSend (
                channel: '#alertas',
                message: "✅ Build exitoso: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
		/*
            // Notificación por correo
            emailext (
                subject: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "El build fue exitoso.",
                to: "destinatario@gmail.com",
                from: "andres.vazquezleon01@gmail.com",
                mimeType: 'text/plain'
            )
            */
        }
        failure {
            slackSend (
                channel: '#general',
                message: "❌ Build fallido: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
		/*
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "El build falló. Verifica los logs.",
                to: "destinatario@gmail.com",
                from: "andres.vazquezleon01@gmail.com",
                mimeType: 'text/plain'
            )
            */
        }
    }
}
