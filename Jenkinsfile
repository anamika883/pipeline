pipeline {
    agent any
    
    stages {
        stage('clone') {
            steps {
                sh 'git clone -b main https://github.com/xaravind/pipeline.git'
             }
       }
       stage('genarate') {
            steps {
                dir("pipeline") {
                    sh 'mvn package'
                    sh 'ls -la'
                }
            }
       }
       stage('deploy') {
            steps {
                dir("pipeline") {
                    sh 'cp -r target/*.war /opt/apache-tomcat-9.0.100/webapps'
                }
            }
       }
    }
}

