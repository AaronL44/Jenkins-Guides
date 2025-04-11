# Jenkins Personal Server Setup and CI/CD Pipeline

## 1. Networking Setup

### Create a VPC
- Custom VPC for isolation.

### Create a Public Subnet
- Public subnet is required because the application is public-facing.

### Create and Attach an Internet Gateway
- Needed to provide internet access to instances in the VPC.

### Configure Security Group (SG)
Allow inbound traffic on:
- **Port 22** (SSH): From your IP or 0.0.0.0/0 for testing
- **Port 80** (HTTP): From 0.0.0.0/0
New port 8080 needed as this is the port Jenkins runs on.
- **Port 8080** (Jenkins): From 0.0.0.0/0

## 2. Launch Jenkins EC2 Instance
- Use **Ubuntu 22.04 LTS** AMI  
- Select **t2.micro** or **t3.micro** instance type  
- Assign it to the **custom VPC** and **public subnet**  
- Enable **Auto-assign Public IP** as this will give the server an ip address inside your VPC that you can use to connect to it once it's set up.

## 3. Install Jenkins on Ubuntu Server

### Connect to Instance
```bash
ssh -i your-key.pem ubuntu@your-instance-public-ip
```

![connecting to server instance](images/connecting%20to%20server%20instance.png)

### Install Java and Jenkins

Next you need to install Java on the Ubuntu instance through the gitbash using the following commands:

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

### Install Jenkins

You now need to install Jenkins on the ubuntu server through the gitbash also. You do this with the following commands:

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]   https://pkg.jenkins.io/debian-stable binary/" | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
### What this code does step by step:

#### 1. Download the Jenkins GPG Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
```

**What it does:**  
Downloads Jenkins' official GPG key and saves it to `/usr/share/keyrings/jenkins-keyring.asc`.  
This key is used to verify the authenticity and integrity of Jenkins packages during installation.

---

#### 2. Add the Jenkins Repository to APT Sources

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

**What it does:**  
Adds the Jenkins Debian repository to your system’s list of APT sources.  
The `signed-by` argument ensures that packages are verified using the key you just downloaded.

---

#### 3. Update Package Lists

```bash
sudo apt-get update
```

**What it does:**  
Refreshes your system's list of available packages and their versions.  
Includes the Jenkins repository added in the previous step.

---

#### 4. Install Jenkins

```bash
sudo apt-get install jenkins
```

**What it does:**  
Installs Jenkins from the newly added Jenkins repository.  
Pulls the latest stable version that is verified by the GPG key.

---

#### 5. Start and Enable Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**What it does:**  
- `start`: Starts the Jenkins service immediately.  
- `enable`: Ensures Jenkins starts automatically on system boot.

---

✅ Jenkins is now installed and running on your Ubuntu server!


## 4. Access Jenkins
- Go to: `http://<instance-public-ip>:8080`
- Retrieve initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
- Install **Suggested Plugins**
- Create admin user (example):
  - Username: `Tech503-aaron`
  - Password: `*****`

![Unlock jenkins page](images/Unlock%20jenkins%20page.png)

Navigate back to your GitBash windows and type the following commands:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

- This will print the initial password for jenkins which will allow you to access the wizard page.

Once you log in select "Install suggested plugins" 
- This will include all default plugins you need. You can also add anything else you need at a later stage.

![Customize Jenkins page](image.png)

Once selected you should see a "Getting Started" page which shows you the plugins being installed on the jenkins server.

![Getting Started](image.png)

Next, a "Create First Admin User" should pop up. Enter a username and password.
- You will use this to login whenever you return to the server in the future.

![First admin user](images/First%20admin%20user.png)

Next, add the Jenkins URL to the instance configuration.
- This is the AWS instance public ip address followed by the port 8080.

![Instance config for jenkins](image.png)

You should now see your Jenkins dashboard after successful access to your jenkins server:

![Jenkins Dashboard](image.png)

## 5. Install NodeJS Plugin
### Why?
- The NodeJS plugin in Jenkins is used to manage and configure specific versions of Node.js for your build jobs. 
- It ensures Jenkins uses a specified Node version for JavaScript build tasks (npm, webpack, etc.), providing consistency across environments.

### How?
- Go to **Manage Jenkins > Tools > Plugins > Available**
- Search and install **NodeJS Plugin**

![NodeJS install](images/NodeJS%20install.png)

- Next go to **Tools** on Jenkins and find the NodeJs plugin. Then configure it to version you need.
- Configure version 20.0 **Global Tool Configuration** 

![Version 20 NodeJs](image.png)

## 6. GitHub SSH Key Host Setup

- The default is **known hosts** but your key isn’t in known hosts folder. 

![Known hosts](images/Known%20hosts.png)

To fix this:
- Add GitHubs known host key to your Jenkins "Approved Host Keys" as this will allow your SSH key connection from GitHub later on.

GitHub public key known hosts found on google:
![github known host google](images/github%20known%20host%20google.png)

Adding the "Known Host" to Jenkins settings known hosts to allow your SSH connection later as it will be a known host.

![Known Host Jenkins](image.png)

## Set up Projects on Jenkins
**Go to CI/CDE Guide for the three projects setup**

## 7. SSH Connection (GitHub and Jenkins)

- Follow SSH section from CI/CDE Guide.
(I didn't have to because it was already set to other jenkins server so i just changed it)

## 8. Create Jenkins Pipeline Projects

### Project 1 Setup:
- Pull from GitHub via SSH
- Configure build steps
- Add post-build action: **Build other projects** (trigger Project 2)

### Test CI:
- Push a change to the repo
- Jenkins builds Project 1 and triggers Project 2

### Project 2 Startup

## 9. Add AWS Deployment Step

### Launch App EC2 Instance from Custom AMI
- Use existing AMI of the app
- Ensure it's launched in the same VPC

### Configure SSH Agent Plugin
- Install **SSH Agent Plugin** if not available
- Add AWS private key via **Manage Credentials**
- In build config, use SSH Agent and remote deploy commands

### Project 2 Setup:
- Add post-build step: **Build Project 3** on success

## 10. Verify Full CI/CD Chain
- Push a change to Project 1 repo
- Trigger build -> Project 2 -> Project 3
- Changes deployed to EC2 app instance

![Change success](images/Change%20success.png)

![Changed home page](images/Changed%20home%20page.png)

## 11. Create Jenkins AMI

### Steps:
- Terminate app instance as it's no longer needed (production environment)
- Stop Jenkins server instance as a snapshot can only be taken when the server instance is stopped.
- Go to **Actions > Create Image**

![Actions image](image.png)

- Wait for the image to run. 
- Launch new instance from AMI to test

## 12. Test the AMI

Terminate the old jenkins server (which you used to create the AMI from)

Launch a new instance using the jenkins AMI.

![Jenkins AMI](images/Jenkins%20AMI.png)

Confirm Jenkins setup is preserved by opening Jenkins on google using the new instance public ip:

![Instance ip](images/Instance%20ip.png)

Use your login (username and password you made earlier) and wait for it to log in.

![Logged in](images/Logged%20in.png)

*AMI Test Successful*

