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
                    dir("${BUILD_NAME}") {
                    sh 'rm -rf ${BUILD_NAME}'
                sh '''
                    git clone -b ${GIT_BRANCH} ${GIT_URL}
                '''
                }
            }
        }
        stage('generate') {
            steps {
                dir("${BUILD_NAME}") {
                    sh 'mvn package'
                    sh 'ls -la target'
                    sh 'mv target/*.war target/${PROJECT_NAME}.${BUILD_ID}.war'
                }
            }
        }
        stage('deploy') {
            steps {
                dir("${BUILD_NAME}") {
                    sh 'cp target/*.war /opt/apache-tomcat-9.0.100/webapps'
                }
            }
        }
        stage('backup') {
            steps {
                dir("${BUILD_NAME}") {
                    sh 'cp target/*.war ${BACKUP}'
                }
            }
        }
        stage('cleanup') {
            steps {
                dir("${BUILD_NAME}") {
                    sh 'rm -rf target/*.war'
                }
            }
        }
    }
}
