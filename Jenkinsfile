pipeline {
    agent any

    stages {
        stage('Build_Maven_Project') {
            steps {
                sh 'mvn archetype:generate -DgroupId=com.example.$Application -DartifactId=$Application -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false'
            }
        }
        stage('Listing_Contents') {
            steps {
                dir ("$Application") {
                sh 'ls -la'
                }
            }
        }
        stage('Building_Application') {
            steps {
                dir ("$Application") {
                sh 'mvn package'
                }
            }
        }
        stage('Deploy_Artifact_To_Tomcat'){
            steps { 
                dir ("$Application") {
                sh 'cp /target/$Application*.war /opt/apache-tomcat-9.0.100/webapps/'
                }
            }
	}
    }
}
