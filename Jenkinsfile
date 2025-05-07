pipeline {
  agent any

  triggers {
    githubPush()
  }

  environment {
    GIT_CREDENTIALS    = 'token_clase'
    SLACK_CREDENTIALS  = 'slack-jenkins'
    SLACK_CHANNEL      = '#alertas'
    EMAIL_RECIPIENTS   = 'andres.vazquezleon01@gmail.com, manuelf.linor@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn clean install'
      }
    }

    stage('Auto Merge to main') {
      when {
        allOf {
          expression { currentBuild.currentResult == 'SUCCESS' }
          expression { env.BRANCH_NAME != 'main' && env.BRANCH_NAME != 'master' }
        }
      }
      steps {
        script {
          def branch = env.BRANCH_NAME
          dir('merge-workspace') {
            git url: scm.userRemoteConfigs[0].url,
                branch: 'main',
                credentialsId: env.GIT_CREDENTIALS

            sh '''
              git config user.name "Andres-Vazque-Leon"
              git config user.email "andres.vazquezleon01@gmail.com"
              git fetch origin
              git merge --no-ff origin/${BRANCH_NAME} -m "Auto-merge ${BRANCH_NAME} into main [ci skip]"
              git push origin main
            '''
          }
        }
      }
    }
  }

  post {
    success {
      slackSend (
        channel: env.SLACK_CHANNEL,
        color: 'good',
        message: "✅ Éxito en ${env.JOB_NAME} #${env.BUILD_NUMBER} en rama ${env.BRANCH_NAME}.\nVer detalles: ${env.BUILD_URL}",
        tokenCredentialId: env.SLACK_CREDENTIALS
      )
      mail to: env.EMAIL_RECIPIENTS,
           subject: "✔️ Pipeline exitoso en ${env.BRANCH_NAME}",
           body: "El build fue exitoso.\n\nVer detalles: ${env.BUILD_URL}"
    }

    failure {
      slackSend (
        channel: env.SLACK_CHANNEL,
        color: 'danger',
        message: "❌ Falló ${env.JOB_NAME} #${env.BUILD_NUMBER} en rama ${env.BRANCH_NAME}.\nVer detalles: ${env.BUILD_URL}",
        tokenCredentialId: env.SLACK_CREDENTIALS
      )
      mail to: env.EMAIL_RECIPIENTS,
           subject: "❌ Falló el pipeline en ${env.BRANCH_NAME}",
           body: "El build falló.\n\nVer detalles: ${env.BUILD_URL}"
    }
  }
}
