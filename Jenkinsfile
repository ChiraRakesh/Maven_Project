pipeline {
    agent any

    tools {
        maven 'Maven'       // Ensure a tool named 'Maven' is configured in Jenkins global tools
        jdk 'OpenJDK11'     // Ensure a JDK tool is configured or rely on system java
    }

    environment {
        // Tomcat URL/APP path
        TOMCAT_URL = 'http://localhost:8080'   // replace with your Tomcat host or IP
        APP_PATH   = '/myapp'                  // context path where WAR will be deployed
    }

    stages {
        stage('Checkout') {
            steps {
                // Use explicit Git checkout instead of checkout scm
                git branch: 'main', url: 'https://github.com/ChiraRakesh/python-cicd-pipeline-demo2.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project with Maven...'
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
                expression { return env.TOMCAT_USE_SCP != 'true' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def war = sh(script: "ls target/*.war | head -n1", returnStdout: true).trim()
                        if (!war) {
                            error "WAR not found in target/"
                        }

                        // Deploy via Tomcat manager text API
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
                sshagent(credentials: ['tomcat-ssh']) {
                    script {
                        def war = sh(script: "ls target/*.war | head -n1", returnStdout: true).trim()
                        sh """
                            scp -o StrictHostKeyChecking=no ${war} tomcat@localhost:/opt/tomcat/webapps/
                            ssh -o StrictHostKeyChecking=no tomcat@localhost 'sudo systemctl restart tomcat'
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Build failed! Email notifications are disabled for now."
            // To enable email, configure SMTP in Jenkins and uncomment the following:
            // mail to: 'dev-team@example.com', subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}", body: "See console output: ${env.BUILD_URL}"
        }
    }
}
