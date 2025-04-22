pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/your-user/your-repo.git'
        ARTIFACT_ID = 'your-artifact'
        BUILD_VERSION = "${env.BUILD_NUMBER}"
        SONARQUBE_ENV = 'SonarQube'              // Name configured under Manage Jenkins > SonarQube Servers
        SONAR_TOKEN = credentials('sonarqube_token')
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Clone Source Code') {
            steps {
                git credentialsId: 'git_creds', url: "${env.GIT_REPO}"
            }
        }

        stage('Build WAR') {
            steps {
                sh "mvn clean package -Drevision=${BUILD_VERSION}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }
    }

    post {
        success {
            echo "Build and SonarQube scan completed."
        }
        failure {
            echo "Build or scan failed."
        }
    }
}
