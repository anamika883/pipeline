pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
        BUILD_NAME = 'pipeline'
        BACKUP = '/opt/'
        PROJECT_NAME = 'DevOps'
    }

    stages {
        stage('clone') {
            steps {
                sh '''
                    git clone -b ${env.GIT_BRANCH} ${env.GIT_URL}
                '''
            }
        }
        stage('generate') {
            steps {
                dir("${env.BUILD_NAME}") {
                    sh 'mvn package'
                    sh 'ls -la target'
                    sh 'mv target/*.war ${env.PROJECT_NAME}.${env.BUILD_ID}.war'
                }
            }
        }
        stage('deploy') {
            steps {
                dir("${env.BUILD_NAME}") {
                    sh 'cp target/*.war /opt/apache-tomcat-9.0.100/webapps'
                }
            }
        }
        stage('backup') {
            steps {
                dir("${env.BUILD_NAME}") {
                    sh 'cp target/*.war ${env.BACKUP}'
                }
            }
        }
        stage('cleanup') {
            steps {
                dir("${env.BUILD_NAME}") {
                    sh 'rm -rf target/*.war'
                }
            }
        }
    }
}
