pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
        MODULE_DIR = 'java_code'
        BACKUP = '/opt/'
        PROJECT_NAME = 'DevOps'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
        TOMCAT_CREDS = credentials('tomcat_credentials')
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
                                             -Dsonar.host.url=http://172.31.19.112:9000 \
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
                        curl -v --user ${NEXUS_CREDS_USR}:${NEXUS_CREDS_PSW} \
                             --upload-file target/${PROJECT_NAME}.${BUILD_ID}.war \
                             http://54.226.147.226:8081/repository/war-repo/${PROJECT_NAME}.${BUILD_ID}.war
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
                    curl -v --user ${TOMCAT_CREDS_USR}:${TOMCAT_CREDS_PSW} \\
                     --upload-file target/${PROJECT_NAME}.${BUILD_ID}.war \\
                     "http://54.226.147.226:8090/manager/text/deploy?path=/${PROJECT_NAME}&update=true"
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
