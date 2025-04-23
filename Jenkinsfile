pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
        MODULE_DIR = 'java_code'
        BACKUP = '/opt/'
        PROJECT_NAME = 'DevOps'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
        SONARQUBE_URL = 'http://54.159.93.31:9000'
        NEXUS_URL = 'http://54.147.143.71:8081/repository/war-files'
        TOMCAT_URL = 'http://54.147.143.71:8090/manager/text/deploy'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Repository checked out to workspace"
            }
        }

        stage('Build WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        mvn clean package
                        ls -la target
                        sudo mv target/*.war target/${PROJECT_NAME}.${BUILD_ID}.war
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube_token')
            }
            steps {
                dir("${MODULE_DIR}") {
                    withSonarQubeEnv('MySonarServer') {
                        sh """
                            mvn sonar:sonar -Dsonar.projectKey=${PROJECT_NAME} \
                                             -Dsonar.host.url=${SONARQUBE_URL} \
                                             -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            environment {
                NEXUS_CREDS = credentials('nexus_credentials')
            }
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        curl -v --user $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW \
                             --upload-file target/'${PROJECT_NAME}.${BUILD_ID}.war' \
                             ${NEXUS_URL}/'${PROJECT_NAME}.${BUILD_ID}.war'
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            environment {
                TOMCAT_CREDS = credentials('tomcat_credentials')
            }
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        curl -v --user ${TOMCAT_CREDS_USR}:${TOMCAT_CREDS_PSW} \
                             --upload-file target/${PROJECT_NAME}.${BUILD_ID}.war \
                             "${TOMCAT_URL}?path=/${PROJECT_NAME}&update=true"
                    """
                }
            }
        }

        stage('Backup WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        sudo cp target/*.war ${BACKUP}
                    """
                }
            }
        }

        stage('Cleanup WAR') {
            steps {
                dir("${MODULE_DIR}") {
                    sh """
                        sudo rm -rf target/*.war
                    """
                }
            }
        }
    }
}
