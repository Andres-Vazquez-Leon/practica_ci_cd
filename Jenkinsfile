pipeline {
  agent any

  /* (Opcional si usas Multibranch: Jenkins tomará la rama automáticamente) */
  triggers {
    githubPush()
  }

  environment {
    GIT_CREDENTIALS    = 'token_clase'      // ID de tus credenciales Git en Jenkins
    SLACK_CREDENTIALS  = 'slack-jenkins'       // ID del token de Slack en Jenkins
    SLACK_CHANNEL      = '#alertas'             // Canal de Slack
    EMAIL_RECIPIENTS   = 'andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com' // Lista de correos, separados por comas
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        // Ajusta al comando que uses, ej. mvn, gradle, npm...
        sh './gradlew clean build'
      }
    }

    stage('Merge to main') {
      when {
        allOf {
          expression { currentBuild.currentResult == 'SUCCESS' }
          expression { env.BRANCH_NAME != 'master' }
        }
      }
      steps {
        script {
          // Clona de nuevo la rama main
          dir('merge-workspace') {
            git url: scm.userRemoteConfigs[0].url,
                credentialsId: GIT_CREDENTIALS,
                branch: 'main'
            // Configura usuario para el merge
            sh '''
              git config user.name "Andres-Vazque-Leon"
              git config user.email "andres.vazquezleon01@gmail.com"
              git merge origin/${BRANCH_NAME}
            '''
            // Empuja el merge
            withCredentials([usernamePassword(
              credentialsId: GIT_CREDENTIALS,
              usernameVariable: 'GIT_USER',
              passwordVariable: 'GIT_PASS'
            )]) {
              sh '''
                git push https://${GIT_USER}:${GIT_PASS}@${scm.userRemoteConfigs[0].url.replace('https://','')} main
              '''
            }
          }
        }
      }
    }
  }

  post {
    success {
      // Notificación Slack
      slackSend(
        channel: SLACK_CHANNEL,
        credentialsId: SLACK_CREDENTIALS,
        color: 'good',
        message: "✅ Éxito: Job *${env.JOB_NAME}* #${env.BUILD_NUMBER} (branch: `${env.BRANCH_NAME}`)\n${env.BUILD_URL}"
      )
      // Notificación correo
      emailext(
        to:      EMAIL_RECIPIENTS,
        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:    """La compilación *fue exitosa* en rama `${env.BRANCH_NAME}`.
Más info: ${env.BUILD_URL}
"""
      )
    }

    failure {
      // No hay merge si falla
      slackSend(
        channel: SLACK_CHANNEL,
        credentialsId: SLACK_CREDENTIALS,
        color: 'danger',
        message: "❌ FALLO: Job *${env.JOB_NAME}* #${env.BUILD_NUMBER} (branch: `${env.BRANCH_NAME}`)\n${env.BUILD_URL}"
      )
      emailext(
        to:      EMAIL_RECIPIENTS,
        subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body:    """La compilación *ha fallado* en rama `${env.BRANCH_NAME}`.
Por favor revisa los logs: ${env.BUILD_URL}
"""
      )
    }
  }
}
