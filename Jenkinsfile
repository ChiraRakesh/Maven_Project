pipeline {
    agent any

    tools {
        jdk 'JDK25'  // Name of the JDK configured in Jenkins (Java 25)
    }

    environment {
        TOMCAT_HOME = "/opt/tomcat"
        APP_NAME = "myapp"
        GIT_REPO = "https://github.com/ChiraRakesh/Maven_Project.git"
    }

    stages {
        stage('Checkout Source') {
            steps {
                echo "Cloning Git repository..."
                git url: "${env.GIT_REPO}", branch: 'main', changelog: true, poll: false
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Building Java application with Maven..."
                sh "mvn clean package"
            }
        }

        stage('Backup Old WAR') {
            steps {
                echo "Backing up previous WAR..."
                sh """
                if [ -f ${TOMCAT_HOME}/webapps/${APP_NAME}.war ]; then
                    mkdir -p ${TOMCAT_HOME}/webapps/backup
                    cp ${TOMCAT_HOME}/webapps/${APP_NAME}.war ${TOMCAT_HOME}/webapps/backup/${APP_NAME}_\$(date +%Y%m%d%H%M%S).war
                    echo "Backup created"
                else
                    echo "No previous WAR found, skipping backup"
                fi
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Deploying WAR to Tomcat..."
                sh """
                cp target/*.war ${TOMCAT_HOME}/webapps/${APP_NAME}.war
                ${TOMCAT_HOME}/bin/shutdown.sh || true
                ${TOMCAT_HOME}/bin/startup.sh
                """
            }
        }
    } // end stages

    post {
        success {
            echo "Deployment Successful! Access your app at http://<server-ip>:8080/${APP_NAME}"
        }
        failure {
            echo "Deployment Failed! Check Jenkins logs for details."
        }
    }
} // end pipeline
