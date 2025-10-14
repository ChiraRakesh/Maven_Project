pipeline {
    agent any

    environment {
        // Tomcat deployment details
        TOMCAT_HOST = "13.222.130.188"       // Your Tomcat server IP
        TOMCAT_PORT = "8080"
        TOMCAT_USER = "jenkins"             // Tomcat Manager username
        TOMCAT_PASS = credentials('tomcat-password') // Jenkins credential ID
        APP_NAME = "myapp"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                // Use your Maven project repository
                git url: 'https://github.com/ChiraRakesh/Maven_Project.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                echo "Building project with Maven..."
                withMaven(maven: 'Maven 3') {
                    sh 'mvn -B clean package'
                }
            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('Deploy to Tomcat (Manager)') {
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
                        echo "Deployment to Tomcat successful!"
                    } catch (err) {
                        error "Tomcat deployment failed: ${err}"
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
            script {
                try {
                    // Optional email notification
                    mail to: 'your.email@example.com',
                         subject: "Jenkins Pipeline Failed: ${currentBuild.fullDisplayName}",
                         body: "Check console output at ${env.BUILD_URL}"
                } catch (err) {
                    echo "Email failed (SMTP not configured): ${err}"
                }
            }
        }
    }
}
