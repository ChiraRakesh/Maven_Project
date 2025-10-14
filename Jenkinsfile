pipeline {
    agent any

    tools {
        // Make sure 'M3' matches your Maven installation name in Jenkins Global Tool Configuration
        maven 'M3'
        jdk 'JDK11' // Use your installed JDK
    }

    environment {
        TOMCAT_USER = credentials('tomcat-user')   // Jenkins credential ID for Tomcat user
        TOMCAT_PASS = credentials('tomcat-pass')   // Jenkins credential ID for Tomcat password
        TOMCAT_URL  = "http://13.222.130.188:8080/manager/text" // Your Tomcat manager URL
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out Git repository...'
                git branch: 'master', url: 'https://github.com/ChiraRakesh/Maven_Project.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project with Maven...'
                // Use withMaven plugin
                withMaven(maven: 'M3') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                script {
                    def warFile = sh(script: "ls target/*.war", returnStdout: true).trim()
                    sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                    -T ${warFile} \
                    ${TOMCAT_URL}/deploy?path=/Maven_Project&update=true
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
