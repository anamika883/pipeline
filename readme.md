
1. **Run SonarQube analysis**  
2. **Upload the WAR to Nexus**  
3. **Deploy the WAR to Tomcat**

---

## 🧩 Breakdown of What You Need to Do

---

### 🔐 1. **Jenkins Credentials Setup**

Go to **Jenkins > Manage Jenkins > Credentials** and add the following credentials:

- **SonarQube Token**  
  - Type: _Secret Text_  
  - ID: `sonarqube_token`

- **Nexus Username/Password**  
  - Type: _Username with password_  
  - ID: `nexus_credentials`

- **Tomcat Manager Credentials**  
  - Type: _Username with password_  
  - ID: `tomcat_credentials`

Make sure the Jenkins user can SSH or reach both servers if needed.

---


---

## 🔐 Add SonarQube Token to Jenkins

### 1. Generate Token in SonarQube
On your SonarQube server (`http://172.31.19.112:9000`):

- Go to your profile → **My Account** → **Security**
- Click **Generate Token**, name it `jenkins-token`
- Copy the token

---

### 2. Add Token in Jenkins
In Jenkins:

- Go to `Manage Jenkins → Credentials → (global) → Add Credentials`
  - **Kind**: `Secret text`
  - **Secret**: *Paste your SonarQube token*
  - **ID**: `sonarqube_token` *(you can use any ID, but we’ll reference this)*
  - **Description**: `SonarQube Token`

---

### 3. Configure SonarQube Server in Jenkins
In `Manage Jenkins → Configure System`:

- Scroll to **SonarQube servers**
- Click **Add SonarQube**
  - **Name**: `MySonarServer`
  - **Server URL**: `http://172.31.19.112:9000`
  - Check **Server authentication token**
    - Choose the credential: `sonarqube_token`
- Save

---




### 🧰 2. **Jenkins Global Tool Configuration**

Go to **Manage Jenkins > Global Tool Configuration**, and:

- Add **SonarQube Scanner** (make sure to name it `SonarQubeScanner`)
- Add your **SonarQube Server** under **Manage Jenkins > Configure System > SonarQube Servers**

---

### 🌐 3. **Ensure These Tools Are Installed**

- Your Maven project must have a `sonar-maven-plugin` in `pom.xml`.
- Nexus repository must be set to accept `WAR` artifacts.
- Tomcat server must have its `/manager/text` endpoint enabled.

---
The error you're getting (`HTTP/1.1 403 Access Denied`) during deployment to Tomcat using the Manager app indicates that your Jenkins pipeline is trying to deploy the WAR file via the **Tomcat Manager text interface**, but the request is being blocked. Here’s what’s likely wrong and how to fix it:

---

### 🔍 Root Cause

The key error message is:
> “**You are not authorized to view this page.**”

And this part clarifies why:
> “**By default the Manager is only accessible from a browser running on the same machine as Tomcat... you'll need to edit the Manager's `context.xml` file.**”

So the issues are likely:
1. **Access restriction based on IP (loopback only)**.
2. **User credentials not having the correct role (`manager-script`)**.

---

#### 1. **Update `conf/tomcat-users.xml`** on your Tomcat server

Make sure the user you're using in Jenkins has the `manager-script` role.

Example:
```xml
<role rolename="manager-script"/>
<user username="jenkins" password="your_password" roles="manager-script"/>
```

> 🔒 Avoid using `manager-gui` for automated tools like Jenkins. Only `manager-script` is needed.

---

#### 2. **Update `webapps/manager/META-INF/context.xml`**

Tomcat restricts remote access to the manager by default. You need to allow Jenkins’ IP.

Find this line:
```xml
<Context ...>
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1"/>
</Context>
```

Change it to allow your Jenkins server's IP (e.g., `54.235.23.249`):
```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1|54\.235\.23\.249"/>
```

> You can use regex ranges to allow multiple IPs, or allow all IPs (not recommended for production):
```xml
allow=".*"
```

---

Let me know if you want help writing that exact snippet into your pipeline!


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



