pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/ChiraRakesh/Maven_Project.git'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'
        TOMCAT_URL = 'http://localhost:8080/manager/text'
        APP_PATH = '/javaapp'
        WAR_NAME = 'javaapp.war'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔹 Cloning from GitHub..."
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build') {
            steps {
                echo "🔹 Building Java application..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Deploying WAR file to Tomcat..."
                sh '''
                curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                     --upload-file target/*.war \
                     "${TOMCAT_URL}/deploy?path=${APP_PATH}&update=true"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful at ${TOMCAT_URL}${APP_PATH}"
        }
        failure {
            echo "❌ Build or Deployment failed."
        }
    }
}
