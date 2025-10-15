pipeline {
    agent any

    environment {
        TOMCAT_HOME = "/opt/tomcat"
        APP_NAME = "myapp"
        PROJECT_DIR = "/opt/projects/JavaApp" // temporary workspace on the Jenkins node
        GIT_REPO = "https://github.com/ChiraRakesh/Maven_Project.git"
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                echo 'Cleaning old workspace...'
                sh """
                rm -rf ${PROJECT_DIR}
                mkdir -p ${PROJECT_DIR}
                """
            }
        }

        stage('Checkout Source') {
            steps {
                echo "Cloning Git repository..."
                git url: "${env.GIT_REPO}", branch: 'main', changelog: true, poll: false
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Building Java application with Maven..."
                sh "mvn clean package -f pom.xml"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Deploying WAR to Tomcat..."
                sh """
                sudo cp target/*.war ${TOMCAT_HOME}/webapps/${APP_NAME}.war
                sudo systemctl stop tomcat
                sudo systemctl start tomcat
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful! Access your app at http://<server-ip>:8080/${APP_NAME}"
        }
        failure {
            echo "Deployment Failed! Check Jenkins logs for details."
        }
    }
}
