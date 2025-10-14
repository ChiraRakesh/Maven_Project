pipeline {
    agent any

    environment {
        // Update these with your real values
        TOMCAT_HOST = "192.168.1.100"      // Replace with your Tomcat server IP or hostname
        TOMCAT_PORT = "8080"
        TOMCAT_USER = "jenkins"            // Tomcat Manager user
        TOMCAT_PASS = credentials('tomcat-password') // Jenkins credential ID
        APP_NAME = "myapp"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/ChiraRakesh/Maven_Project.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                // Use Maven installed in Jenkins
                withMaven(maven: 'Maven 3') {
                    sh 'mvn clean package'
                }
            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = "target/webapp.war"
                    if (!fileExists(warFile)) {
                        error "WAR file not found: ${warFile}"
                    }

                    try {
                        sh """
                        curl --fail --upload-file ${warFile} \
                        http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/${APP_NAME}&update=true \
                        --user ${TOMCAT_USER}:${TOMCAT_PASS}
                        """
                        echo "Deployment successful!"
                    } catch (err) {
                        error "Deployment failed: ${err}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed."
            // Optional email notification
            mail to: 'your.email@example.com',
                 subject: "Jenkins Pipeline Failed: ${currentBuild.fullDisplayName}",
                 body: "Check console output at ${env.BUILD_URL}"
        }
    }
}
