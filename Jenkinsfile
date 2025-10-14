pipeline {
  agent any

  tools {
    maven 'Maven'      // Ensure a tool named 'Maven' is configured in Jenkins global tools
    jdk 'OpenJDK11'    // Ensure a JDK tool configured or rely on system java
  }

  environment {
    # Tomcat URL/APP path
    TOMCAT_URL = 'http://tomcat.example.com:8080'   // replace with your Tomcat host (or http://localhost:8080 if same host)
    APP_PATH = '/myapp'                             // context path where WAR will be deployed (/myapp -> myapp.war)
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
      }
    }

    stage('Deploy to Tomcat (manager)') {
      when {
        expression { return env.TOMCAT_USE_SCP != 'true' } // default: use manager; set TOMCAT_USE_SCP=true to use scp method
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
          script {
            def war = sh(script: "ls target/*.war | head -n1", returnStdout: true).trim()
            if (!war) {
              error "WAR not found in target/"
            }
            // Deploy via Tomcat manager text API: update if exists
            sh """
              set -e
              curl --fail --upload-file ${war} \\
                   "${TOMCAT_URL}/manager/text/deploy?path=${APP_PATH}&update=true" \\
                   --user "${TOMCAT_USER}:${TOMCAT_PASS}"
            """
          }
        }
      }
    }

    stage('Deploy to Tomcat (scp)') {
      when {
        expression { return env.TOMCAT_USE_SCP == 'true' }
      }
      steps {
        // requires SSH credentials added as 'tomcat-ssh' in Jenkins
        sshagent (credentials: ['tomcat-ssh']) {
          script {
            def war = sh(script: "ls target/*.war | head -n1", returnStdout: true).trim()
            sh """
              scp -o StrictHostKeyChecking=no ${war} tomcat@tomcat.example.com:/opt/tomcat/webapps/
              ssh -o StrictHostKeyChecking=no tomcat@tomcat.example.com 'sudo systemctl restart tomcat'
            """
          }
        }
      }
    }
  }

  post {
    failure {
      mail to: 'dev-team@example.com', subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}", body: "See console output: ${env.BUILD_URL}"
    }
  }
}
