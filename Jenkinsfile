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
        // Usa el comando apropiado para tu stack (por ejemplo npm, mvn...)
        sh 'mvn clean install'
      }
    }

    stage('Merge if PR or branch') {
      when {
        allOf {
          expression { currentBuild.currentResult == 'SUCCESS' }
          anyOf {
            expression { env.CHANGE_ID != null }           // PR
            expression { env.BRANCH_NAME != 'main' && env.BRANCH_NAME != 'master' } // Rama distinta de main
          }
        }
      }
      steps {
        script {
          def branchToMerge = env.CHANGE_ID != null ? env.CHANGE_BRANCH : env.BRANCH_NAME

          sh """
            git config user.name "jenkins-bot"
            git config user.email "jenkins@ci"

            git remote set-url origin https://${env.GIT_CREDENTIALS}@github.com/${env.GIT_URL.replaceFirst('https://github.com/', '')}
            git fetch origin master
            git checkout master
            git merge --no-ff origin/${branchToMerge} -m "Auto merge from ${branchToMerge} via Jenkins"
            git push origin master
          """
        }
      }
    }
  }

  post {
    failure {
      slackSend (
        channel: env.SLACK_CHANNEL,
        color: 'danger',
        message: "❌ Falló el build para ${env.BRANCH_NAME}.",
        tokenCredentialId: env.SLACK_CREDENTIALS
      )
      emailext (
        subject: "Build fallido: ${env.JOB_NAME}",
        body: "Revisa el error: ${env.BUILD_URL}",
        to: env.EMAIL_RECIPIENTS
      )
    }
  }
}
