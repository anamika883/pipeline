pipeline {
    agent any

     environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
	PROJ_NAME= 'pipeline'
	DEPLY
    }

    stages {
        stage('clone') {
            steps {
                sh '''
		git clone -b ${env.GIT_BRANCH} ${env.GIT_URL}
		'''
             }
       }
       stage('genarate') {
            steps {
                dir("$PROJ_NAME") {
                    sh 'mvn package'
                    sh 'ls -la'
                }
            }
       }
       stage('deploy') {
            steps {
                dir("$PROJ_NAME") {
                    sh 'cp -r target/*.war /opt/apache-tomcat-9.0.100/webapps'
                }
            }
       }
    }
}

