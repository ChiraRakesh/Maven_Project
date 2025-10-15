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
