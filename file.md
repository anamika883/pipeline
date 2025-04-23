# CI/CD Pipeline Setup with Jenkins, Nexus, Tomcat, and SonarQube

## Infrastructure Setup

Created three EC2 servers with `t2.medium` instance type:

1. **jenkins_build**
2. **nexus_tomcat**
3. **sonarqube**

![Image](https://github.com/user-attachments/assets/3590e514-2c81-477b-8df0-08534398ae53)

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
[ec2-user@ip-172-31-23-82 install_scripts]$ ./jenkins_build.sh
[INFO] Re-running script as root...
========== Script started at Wed Apr 23 08:51:14 UTC 2025 ==========
[INFO] Updating system...
..
..
[SUCCESS] Jenkins started successfully!
[INFO] Installation completed successfully!
```

Once completed, Jenkins, Nexus, Tomcat, and SonarQube will be running.

Verify:
```bash
curl ifconfig.me
netstat -ntlp
```
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
Sample output:
```bash
[ec2-user@ip-172-31-23-82 install_scripts]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
1853f621edb045a68ad11c20c0d478e5
```
2. Copy the password and follow the setup wizard in the browser.
3. Set username, password, and install suggested plugins.

![Image](https://github.com/user-attachments/assets/de4337f2-6b52-48b2-ac44-0e750b48584a)

![Image](https://github.com/user-attachments/assets/229bb58f-9cff-4fc4-9821-cc1542eeb22d)

![Image](https://github.com/user-attachments/assets/bff05732-e628-4b5a-9ac9-5837b569718c)

![Image](https://github.com/user-attachments/assets/e2455d91-5d49-43f3-8fab-161286956b22)

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
Sample output:
```bash
[ec2-user@ip-172-31-19-48 install_scripts]$ sudo sudo cat /opt/sonatype-work/nexus3/admin.password
6f412470-d4fa-4bb1-949e-df250c2594b9
```

2. Username: `admin` and password: `admin`
3. Set a new password, configure anonymous access, and finish setup.

![Image](https://github.com/user-attachments/assets/bdc3f3ad-574b-4bab-885c-847574f9a33b)

![Image](https://github.com/user-attachments/assets/eb3f90b0-23d8-48fd-831a-f8fa719890fa)

![Image](https://github.com/user-attachments/assets/dcd59413-b613-4949-90f2-3aadf7eefbf1)

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
![Image](https://github.com/user-attachments/assets/4222e9da-891c-4f3b-9a08-179592571737)

![Image](https://github.com/user-attachments/assets/d4399891-99a3-4ff2-a0ba-031451a887ec)

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
![Image](https://github.com/user-attachments/assets/ca983f22-3e2c-471d-8ead-54043360daf0)

![Image](https://github.com/user-attachments/assets/dcb2f369-7a16-4cb8-9927-5c076aca2987)

![Image](https://github.com/user-attachments/assets/876e446d-d497-40d5-8340-5ec68d0c48fc)

![Image](https://github.com/user-attachments/assets/826e5f14-ded7-42ef-8119-9aa870a9b34b)

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

![Image](https://github.com/user-attachments/assets/ce72d15f-50c8-4460-8aba-8b88e692e456)

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
   
![Image](https://github.com/user-attachments/assets/84c7d2c3-1108-47e0-a0f1-01a631c65526)

![Image](https://github.com/user-attachments/assets/da424622-bc02-4c5f-9dbc-7a763e9330d6)

![Image](https://github.com/user-attachments/assets/68292173-f577-4a44-92b8-403aff6b3f64)

### Trigger Build
- Add a blank line at the end of the Jenkinsfile and push the changes
- 
![Image](https://github.com/user-attachments/assets/5b466e55-a40e-4829-b740-8218975d3fcd)

---
![Image](https://github.com/user-attachments/assets/04e3e197-f837-496e-ba68-d5c94e231214)



## CI/CD Pipeline Flow

1. Build the WAR file
2. Run SonarQube analysis 
3. Upload to Nexus repository
4. Deploy WAR to Tomcat
5. Backup WAR locally
6. Cleanup WAR from code folder

![Image](https://github.com/user-attachments/assets/5ea933ef-17a0-479b-a299-40bf1870568a)

![Image](https://github.com/user-attachments/assets/396e5493-349c-4a4a-b8fb-70a4c2d93f28)

![Image](https://github.com/user-attachments/assets/4f751772-60ec-41f5-bf44-0a7064c5a39c)

![Image](https://github.com/user-attachments/assets/2166138a-2fa3-4fc9-a6db-360fe8486319)

---


