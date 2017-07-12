pipeline {
  agent {
    docker {
      image 'gradle'
    }
  }
  stages {
    stage('build') {
      steps {
        sh '''
gradle --no-daemon -g "`pwd`/.gradle" clean
gradle --no-daemon -g "`pwd`/.gradle" distZip
'''
      }
    }
    stage('test') {
      steps {
        sh '''
gradle --no-daemon -g "`pwd`/.gradle" test -i
'''
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/build/test-results/**/*.xml'
        }
      }
    }
    stage('archive') {
      when {
        branch 'master'
      }
      steps {
        archiveArtifacts '**/build/distributions/*.zip'
        emailext to: 'w@vecsight.com,t@vecsight.com,p@vecsight.com',
          body: 'Artifacts:',
          subject: "ARTIFACTS: Pipeline '${env.JOB_NAME}' ${env.BUILD_DISPLAY_NAME}",
          attachmentsPattern: '**/build/distributions/*.zip'
      }
    }
    stage('deploy') {
      when {
        branch 'master'
      }
      steps {
        script {
          try {
            mail to: 'w@vecsight.com,t@vecsight.com',
              mimeType: 'text/html',
              subject: "DEPLOYMENT CONFIRM: Pipeline '${env.JOB_NAME}' ${env.BUILD_DISPLAY_NAME}",
              body: "<a href=\"${env.BUILD_URL}input\">Click here to proceed or abort</a><br><br>Or ${env.BUILD_URL}input"
            timeout(time: 1, unit: 'HOURS') {
              input message: 'Deploy?'
            }
            echo 'deploying to yoshino'
            sshagent(['ssh_yoshino']) {
              sh 'scp -o StrictHostKeyChecking=no ./dragonite-forwarder/build/distributions/dragonite-forwarder*.zip tobyxdd@yoshino.vecsight.com:/home/tobyxdd/jenkins/dragonite-forwarder.zip'
              sh 'ssh -o StrictHostKeyChecking=no tobyxdd@yoshino.vecsight.com "cd /home/tobyxdd/jenkins/; bash dragonited.sh"'
            }
            echo 'deploying to batman'
            sshagent(['ssh_batman']) {
              sh 'scp -o StrictHostKeyChecking=no ./dragonite-forwarder/build/distributions/dragonite-forwarder*.zip tobyxdd@batman.vecsight.com:/home/tobyxdd/jenkins/dragonite-forwarder.zip'
              sh 'ssh -o StrictHostKeyChecking=no tobyxdd@batman.vecsight.com "cd /home/tobyxdd/jenkins/; bash dragonited.sh"'
            }
          } catch (err) {
            echo 'deployment aborted'
          }
        }
      }
    }
  }
  post {
    failure {
      script {
        if (env.BRANCH_NAME == 'master') {
          emailext to: 'w@vecsight.com,t@vecsight.com',
            mimeType: 'text/html',
            subject: "FAILURE: Pipeline '${env.JOB_NAME}' ${env.BUILD_DISPLAY_NAME}",
            body: "<a href=\"${env.BUILD_URL}\">Click here for more detail</a><br><br>Or ${env.BUILD_URL}",
            attachLog: true
        }
      }
    }
  }
}
