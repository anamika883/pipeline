pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
        MODULE_DIR = 'java_code'
        BACKUP = '/opt/'
        PROJECT_NAME = 'DevOps'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                // If using Pipeline script from SCM, Jenkins already does checkout
                echo "Repository checked out to workspace"
            }
        }

        stage('Build WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        mvn clean package
                        ls -la target
                        mv target/*.war target/${PROJECT_NAME}.${BUILD_ID}.war
                    """
                }
            }
        }

        stage('Backup WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        cp target/*.war ${BACKUP}
                    """
                }
            }
        }

        stage('Cleanup WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        rm -rf target/*.war
                    """
                }
            }
        }
    }
}
