# CI/CD Pipeline Setup with Jenkins, Nexus, Tomcat, and SonarQube

## Infrastructure Setup

Created three EC2 servers with `t2.medium` instance type:

1. **jenkins_build**
2. **nexus_tomcat**
3. **sonarqube**

### Login to Each Server
Update and install required packages with `install_scripts`

#### For jenkins_build:
```bash
sudo yum update -y
sudo yum install git -y
git clone https://github.com/xaravind/install_scripts.git
```
Sample output:
```bash
[ec2-user@ip-172-31-23-82 ~]$ cd install_scripts/
[ec2-user@ip-172-31-23-82 install_scripts]$ ll
total 12
-rw-r--r--. 1 ec2-user ec2-user 1732 Apr 23 08:46 jenkins_build.sh
-rw-r--r--. 1 ec2-user ec2-user 3256 Apr 23 08:46 nexus_tomcat.sh
-rw-r--r--. 1 ec2-user ec2-user 1786 Apr 23 08:46 sonarqube.sh
```

Give execute permission and run the respective script:
```bash
chmod 755 *.sh
sh jenkins_build.sh
```
Sample output:
```bash
[INFO] Jenkins started successfully!
```

Once completed, Jenkins, Nexus, Tomcat, and SonarQube will be running.

Verify:
```bash
curl ifconfig.me
netstat -ntlp
```

Access via browser:
- Jenkins: `http://<public-ip>:8080`
- Nexus: `http://<public-ip>:8091`
- Tomcat: `http://<public-ip>:8080`
- SonarQube: `http://<public-ip>:9000`

---

## Jenkins Setup

### Initial Setup
1. Run:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
2. Copy the password and follow the setup wizard in the browser.
3. Set username, password, and install suggested plugins.

### Add Sudo Permissions to Jenkins
```bash
sudo visudo
```
Add:
```bash
jenkins ALL=(ALL) NOPASSWD: ALL
```

---

## Nexus Setup

1. Login using initial password:
```bash
sudo cat /opt/sonatype-work/nexus3/admin.password
```
2. Username: `admin`
3. Set a new password, configure anonymous access, and finish setup.

---

## Tomcat Setup

### tomcat-users.xml
Update `conf/tomcat-users.xml`:
```xml
<role rolename="manager-script"/>
<user username="jenkins" password="your_password" roles="manager-script"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="your_password" roles="manager-gui"/>
```

### context.xml
Update `webapps/manager/META-INF/context.xml`:
```xml
<Context>
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1"/>
</Context>
```

---

## SonarQube Setup

1. Login with default credentials:
   - Username: `admin`
   - Password: `admin`
2. Update password.
3. Generate a token:
   - My Account → Security
   - Name: `jenkins-token`
   - Type: Global analysis token
   - Save the token

---

## Jenkins Plugin Installation

1. Go to: `Manage Jenkins → Plugins → Available`
2. Install **SonarQube Scanner** plugin (version 2.18)
3. Restart Jenkins

---

## Jenkins Credentials Setup

Go to: `Manage Jenkins → Credentials → (global) → Add Credentials`

- **SonarQube Token**:
  - Kind: `Secret text`
  - Secret: *Paste your token*
  - ID: `sonarqube_token`

- **Nexus Credentials**:
  - Kind: `Username with password`
  - ID: `nexus_credentials`

- **Tomcat Manager Credentials**:
  - Kind: `Username with password`
  - ID: `tomcat_credentials`

---

## Jenkins Configuration for SonarQube

### SonarQube Servers
Go to: `Manage Jenkins → Configure System → SonarQube Servers`
- Name: `MySonarServer`
- URL: `http://<sonarqube-ip>:9000`
- Choose `sonarqube_token`
- Save

### Global Tool Configuration
Go to: `Manage Jenkins → Global Tool Configuration`
- Add **SonarQube Scanner** with name `SonarQubeScanner`

---

## GitHub Webhook Setup

Go to your GitHub repo → Settings → Webhooks → Add webhook:
- Payload URL: `http://<jenkins-public-ip>:8080/github-webhook/`
- Content type: `application/json`
- Save

Sample repo with Jenkinsfile and Java code: [https://github.com/xaravind/pipeline.git](https://github.com/xaravind/pipeline.git)

---

## Create Jenkins Pipeline

1. Go to Jenkins → New Item → Name: `BuildWarPipeline` → Type: `Pipeline`
2. Configure:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repo URL: *your GitHub URL*
   - Branch: *your branch*
   - Jenkinsfile location: root or path to Jenkinsfile
3. Save

### Trigger Build
- Add a blank line at the end of the Jenkinsfile and push the changes

---

## CI/CD Pipeline Flow

1. Build the WAR file
2. Run SonarQube analysis 
3. Upload to Nexus repository
4. Deploy WAR to Tomcat
5. Backup WAR locally
6. Cleanup WAR from code folder

---


