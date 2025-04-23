
Created three ec2-servers with t2.medium

1. jenkins_build
2. Nexus_tomcat
3. SonarQube

Login to each server

Update and install required package with install_scripts

1. jenkins_build

sudo yum update -y
sudo yum git -y
git clone https://github.com/xaravind/install_scripts.git

you'll something like below output

```bash
[ec2-user@ip-172-31-23-82 ~]$ cd install_scripts/
[ec2-user@ip-172-31-23-82 install_scripts]$ ll
total 12
-rw-r--r--. 1 ec2-user ec2-user 1732 Apr 23 08:46 jenkins_build.sh
-rw-r--r--. 1 ec2-user ec2-user 3256 Apr 23 08:46 nexus_tomcat.sh
-rw-r--r--. 1 ec2-user ec2-user 1786 Apr 23 08:46 sonarqube.sh
```
Give execute permission and run the according to server, it will install and make tools up and running.

chmod 755 *.sh
sh jenkins_build.sh

sample output

```bash
[ec2-user@ip-172-31-23-82 install_scripts]$ ./jenkins_build.sh
[INFO] Re-running script as root...
========== Script started at Wed Apr 23 08:51:14 UTC 2025 ==========
[INFO] Updating system...
Last metadata expiration check: 0:05:50 ago on Wed Apr 23 08:45:24 2025.
Dependencies resolved.
Nothing to do.
Complete!
[INFO] Installing Amazon Corretto 21 JDK...
Complete!
[INFO] Reloading systemd daemon...
[INFO] Starting Jenkins...
Created symlink /etc/systemd/system/multi-user.target.wants/jenkins.service → /usr/lib/systemd/system/jenkins.service.
[INFO] Checking Jenkins status...
[SUCCESS] Jenkins started successfully!
[INFO] Installation completed successfully!
========== Script ended at Wed Apr 23 08:51:48 UTC 2025 ==========
[ec2-user@ip-172-31-23-82 install_scripts]$

```

at the end all of your jenkins, nexus , sonar and tomcat will be up and running

```bash
[ec2-user@ip-172-31-23-82 install_scripts]$ curl ifconfig.me
54.221.142.241[ec2-user@ip-172-31-23-82 install_scripts]$ netstat -ntlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -
tcp6       0      0 :::8080                 :::*      
```
access them with public ip and port no:- 

jenkins - http://<public-ip>:8080
ex:- http://54.221.142.241:8080/
Nexus - http://<public-ip>:8091
ex:- http://54.147.143.71:8091/
tomcat - http://<public-ip>:8080
ex:- http://54.147.143.71:8080/
sonarqube - http://<public-ip>:9000
ex:- http://54.159.93.31:9000/


## Login to jenkins and setup password

1. [ec2-user@ip-172-31-23-82 install_scripts]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword

1853f621edb045a68ad11c20c0d478e5

2. copy and paste the code in browser, click on continue, select install required plugins
 give required details and set up username and password, click on save and continue, next click on save and finish.

 3. click on start using jenkins, jenkins server is ready.

## setup Nexus

1. click on sign in 

go to nexus_tomcat server and get intial password

sudo cat /opt/sonatype-work/nexus3/admin.password


[ec2-user@ip-172-31-19-48 install_scripts]$ sudo sudo cat /opt/sonatype-work/nexus3/admin.password
6f412470-d4fa-4bb1-949e-df250c2594b9

username - admin
password - paste password and signin

2. it will login and pop up welcome tab click on next give new password , click on next , click on agree, select Configure Anonymous Access, click on next and finish

3. nexus server is ready!!

## setup tomcat

configure tomcat-users.xml and context.xml file to setup user and password and to access the server.

1. Update `conf/tomcat-users.xml`** on your Tomcat server

Make sure the user you're using in Jenkins has the `manager-script` role.

Example:
```xml
<role rolename="manager-script"/>
<user username="jenkins" password="your_password" roles="manager-script"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="your_password" roles="manager-gui"/>
```

2. Update `webapps/manager/META-INF/context.xml`**

Tomcat restricts remote access to the manager by default. You need to allow Jenkins’ IP.

Find this line:
```xml
<Context ...>
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1"/>
</Context>
```

Tomcat setup ready!!!

## setup sonarqube

1. Login with default username and password 
admin and admin


2. setup new password

2. Generate Token in SonarQube
On your SonarQube server (`http://54.159.93.31:9000/`):

- Go to your profile → **My Account** → **Security**
-  name it `jenkins-token`
- select Type as Global anaylsis token
-  Click **Generate Token**,
- Copy the token


Sonarqube  server Ready!!!

---
### jenkins plugin installation

Go to **Jenkins > Manage Jenkins > plugins > Availiable pligins**

search SonarQube ScannerVersion 2.18, select and install and restart jenkins server




### 🔐 1. **Jenkins Credentials Setup**

In Jenkins:

- Go to `Manage Jenkins → Credentials → (global) → Add Credentials`
  - **Kind**: `Secret text`
  - **Secret**: *Paste your SonarQube token*
  - **ID**: `sonarqube_token` *(you can use any ID, but we’ll reference this)*
  - **Description**: `SonarQube Token`

similar way update below passwords

- **Nexus Username/Password**  
  - Type: _Username with password_  
  - ID: `nexus_credentials`

- **Tomcat Manager Credentials**  
  - Type: _Username with password_  
  - ID: `tomcat_credentials` # manager-script  - user name and password



### 3. Configure SonarQube Server in Jenkins
In `Manage Jenkins → Configure System`:

- Scroll to **SonarQube servers**
- Click **Add SonarQube**
  - **Name**: `MySonarServer`
  - **Server URL**: `http://<public-ip>:9000`
  - Check **Server authentication token**
    - Choose the credential: `sonarqube_token`
- Save

---

### 🧰 2. **Jenkins Global Tool Configuration**

Go to **Manage Jenkins > Global Tool Configuration**, and:

- Add **SonarQube Scanner** (make sure to name it `SonarQubeScanner`)
- Add your **SonarQube Server** under **Manage Jenkins > Configure System > SonarQube Servers**

---

## setup webhook to your github repo

go to your repo, select settings--> webhooks --> 
give payload url like below
http://<jenkins-public-ip>/:8080/github-webhook/

select content type as application/json --> save

my pipeline githubrepo for your refrence

https://github.com/xaravind/pipeline.git

that will have Jenkinsfile and a java code 

## create a pipeline

click on new item , give name - BuildWarPipeline , select pipeline , click on ok

go to pipeline on left hand side

select pipeline script from scm 

seleect scm as git

give your repo url that webhooks configured and contains jenkinfile and code

give your branch name 

click on save

for job trigger just add a empty  line at the end of jenkinsfile,

add, commit and push so it will trigger the build





## 🚀 Final Enhanced Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/xaravind/pipeline.git'
        GIT_BRANCH = 'main'
        MODULE_DIR = 'java_code'
        BACKUP = '/opt/'
        PROJECT_NAME = 'DevOps'
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
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
                             http://172.31.18.185:8081/repository/war-repo/${PROJECT_NAME}.${BUILD_ID}.war
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
                             "http://172.31.18.185:8080/manager/text/deploy?path=/${PROJECT_NAME}&update=true"
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
```

---



