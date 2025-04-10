# Guide to setting Up Continuous Integration (CI) & Continuous Deployment (CDE) in Jenkins
![Jenkind CI/CD](images/CI-CD%20Diagram.png)

## CI/CD (e) Explained:

### Continuous Integration (CI)
- **Definition**: CI is the practice of automatically testing and integrating code changes whenever developers push to a shared repository.
- **Key Benefits**:
  - Encourages developers to push code **early and often**.
  - Increases **development speed** by allowing quick integration of individual contributions.
  - Ensures the codebase remains **stable and testable** throughout development.

---

### Continuous Delivery vs. Continuous Deployment

| Term | Description |
|------|-------------|
| **Continuous Delivery (CD)** | Code changes from CI are automatically prepared for release. The code is always in a **deployable state**, but deployment requires **manual approval** (e.g., pressing a "big red button"). |
| **Continuous Deployment (CDE)** | Code changes are automatically tested and **released to end users** without any manual intervention—**no big red button**. If all tests pass, the code is deployed automatically. |

---

### Why Companies Choose CD Over CDE

- **Manual control**: CD allows teams to review with stakeholders or align releases with marketing, business, or compliance schedules.
- **Scheduled releases**: Some features or updates are planned for release on specific dates.
- **Risk management**: Automatic deployment (CDE) is often used for small, low-risk changes like bug fixes, whereas larger or more sensitive updates go through CD.

---

### Benefits of CI/CD

- Faster release of **new features and updates** (e.g., in games or apps).
- Higher **developer productivity**.
- Improved **code quality and reliability** through automated testing and consistent deployment processes.

## Pros of Jenkins

1. **Open Source & Free**  
   Completely free to use, with strong community support and regular updates.

2. **Highly Customisable**  
   Supports 1,800+ plugins for integrating with virtually any tool (e.g., GitHub, Docker, Kubernetes, Slack).

3. **Platform Independent**  
   Runs on Windows, macOS, and Linux. Can also be run in containers (e.g., Docker) or in the cloud.

4. **Strong Support for CI/CD Pipelines**  
   Supports complex workflows via Jenkins Pipelines (written in Groovy) and also simpler freestyle jobs.

5. **Large Community**  
   Lots of documentation, tutorials, and forums for support.

6. **Scalable**  
   Can distribute builds across multiple machines (master-agent architecture) to speed up large builds and tests.

7. **Active Ecosystem**  
   Easily integrates with popular development, testing, and deployment tools.

---

## Cons of Jenkins

1. **Steep Learning Curve**  
   Especially when dealing with scripted pipelines, plugins, and advanced configurations.

2. **UI Can Be Clunky**  
   The web interface is dated and not always intuitive, especially compared to newer CI/CD tools.

3. **Plugin Maintenance**  
   Heavy reliance on plugins means frequent updates—and sometimes plugins break or conflict with each other.

4. **Configuration Complexity**  
   Setting up and maintaining Jenkins, especially in large teams or enterprise environments, can become complex and time-consuming.

5. **Limited Built-in Analytics**  
   Lacks advanced built-in analytics or dashboards without third-party tools or plugins.

6. **Manual Scaling Challenges**  
   While Jenkins can be scaled, it often requires manual setup (e.g., adding agents, managing load distribution).

---

## Typical Stages of a Jenkins Pipeline

1. **SCM** – Source Code Management (e.g., Git) to track and manage code changes.
2. **Build** – Compiling the source code and dependencies into an executable format.
3. **Test** – Running automated or manual tests to ensure code functions as expected.
4. **Package** – Bundling the code and dependencies into deployable units (e.g., `.tar`, `.zip`).
5. **Deploy** – (In CDE) Automatically deploy the application.
6. **Monitor** – Continuously track application performance and health in production.

---

## Competitors to Jenkins

- **GitHub Actions**
- **GitLab CI/CD**
- **Azure DevOps**
- **Spinnaker**


# Step By Step Guide:

## Step 1: Continuous Integration (CI) implementation

### Step 1.1: Link jenkins to github repo using SSH keys

### Step 1.1.1: Setup SSH public key on github

First you need to generate an SSH key pair for this repo that jenkins will use to connect with.
- This is good for security and very good for automation as we can give it to a tool like Jenkins and without our intervention it can unlock the resource without our manual input. 
- This is how it will go and get our source code form our repo. 

Open a gitbash window on your local machine and naviagte to the .ssh directory.

Next, type the following command to generate an SSH key pair:
`ssh-keygen -t rsa -b 4096 -C <"your email address">`
- Make sure you use your email address.

Choose a name for your key pair to differentiate them from previous keys.

![SSH key pair name](images/SSH%20key%20pair%20name.png)

Skip the password (unless you want to use one).

Now `ls` to check if the key pair was created:

![Key pair created](images/Key%20pair%20created.png)

Navigate to github select your repo's settings at the top of your repo page (not your github settings)

On the  left side of your settings page select "Deploy Keys"

Select "add new" and choose a name.

Open your gitbash on your local machine and run the following command to print your public key:
`cat <name of your key.pub>`

Then navigate back to github and paste that public key into the key text box:

![Public key](images/Public%20key.png)

Ensure you check "Allow write access"

Finally select "add key".

![Key added to github](images/Key%20added%20to%20github.png)


### Step 1.1.2: Setup private key on jenkins

Select "New item" on the left side of your jenkins dashboard and enter a name.

Select the freestyle project type as you did in the earlier test projects.

Choose a meaningful description for the project:

![CI project description](images/CI%20project%20description.png)

Check the "Discard old builds" setting again and set the "max # of builds to keep" as 3.
- Again this means that the project will only keep the most recent three builds. 
- Specifically making sure to keep the last successful build and then the last 3 builds.
- Note:
  - It's not getting rid of compile files but more the build history and configuration used.

Check the "GitHub project" option as this project will be linked to github. 

Navigate to your github repo tab on google and copy the url. Paste this url in the "Project url" textbox:

![github url for project url](images/github%20url%20for%20project%20url.png)
![Project url textbox](images/Project%20url%20textbox.png)

Next, scroll to the "Source Code Management" (SCM) and check the "git" option. 

Navigate back to your github repo and copy the SSH url from the "code" section.

![SSH github url](images/SSH%20github%20url.png)

Paste that url inside the "repository URL" on jenkins:

![SCM repo url](images/SCM%20repo%20url.png)

Now add the key in the "credentials" drop down menu

![SSH usernamer with private key credentials](images/SSH%20usernamer%20with%20private%20key%20credentials.png)

Add a name, description and username to it:

![Name, description and username of priv key](images/Name,%20description%20and%20username%20of%20priv%20key.png)

Ensure you navigate to your gitbash window on your local machine and run the following command inside your .ssh directory
`cat <name of private key>`

Copy the entire private key including: 
- "-----BEGIN OPENSSH PRIVATE KEY-----" and the:
- -----END OPENSSH PRIVATE KEY-----
- **DO NOT SHARE THIS KEY WITH ANYONE**.

Paste it into the private key textbox:

![Private key text box jenkins](images/Private%20key%20text%20box%20jenkins.png)

Next, change the "Branch Specifier (blank for 'any')" to "*/main"

![Branch Specifier](images/Branch%20Specifier.png)
- We change the branch specifier in Jenkins to match the exact branch we want to build (e.g., main) when using a private SSH key, ensuring Jenkins pulls the correct code from the private repo using the right authentication method.

Navigate to the "Build steps" and select "Execute shell". In here type the following command `ls`.
- This will test whether the connection between jenkins and your repo on github was successful as it will list the files in your GitHub repo in the "console output".

Save the changes and then select "build" to build the project you jsut created. 

Lastly, navigate to the built project once it shows as successfully built (green tick). 

Select "console output" and check the output. It should print the ls command and show the contents from your github repo. 
- In my case it should show the "app copy"

![Console output after jenkins and github connection check](images/Console%20output%20after%20jenkins%20and%20github%20connection%20check.png)

*jenkins to github link success with SSH keys*

### Step 1.2: Adding NodeJs to jenkins

It is now necessary to install NodeJs on Jenkins.

Navigate to the CI project configuration settings and select the "Provide Node & npm bin/ folder to PATH" checkbox:

![nodejs checkbox](images/nodejs%20checkbox.png)

Leave everything else as the default settings.

Navigate to the "Build steps" section and this to the "execute shell":
```` 
ls 
cd app
npm install
npm test

````
- These steps navigate to your app, install dependencies, and run tests—a typical part of a CI job in Jenkins.

Click "save" to save the changes and click build to test it again. 

*NodeJs installed*

### Step 1.3: Create a WEBHOOK

Navigate to the configure settings inside the CI project again.

Change the "Branch Specifier (blank for 'any')" from "*/main" to "*/dev"

![Branch specifier dev](images/Branch%20specifier%20dev.png)

Scroll to the "Build Triggers" and select the "Github hook trigger for GITScm polling"

![BUILD TRIGGERS GITSCM](images/BUILD%20TRIGGERS%20GITSCM.png)
- This tells github to listen for a webhook.

Next, move to your github tab on google.

Select "add webhook" 

![Webhook link github](images/Webhook%20link%20github.png)

Copy your CI project url from jenkins tab and paste it into the "Payload URL" on github. 

Ensure you add "/github-webhook/" to the end of the url you paste into github.

![Payload url](images/Payload%20url.png)

Leave the content type as it is and no secret.

Ensure you "Disable" SSL verification as this is only for test purposes. 
- You would enable this if you were in production and not testing.

Select "Just the push event" and select the "Active" checkbox.

![Disable SSL](images/Disable%20SSL.png)

Select "add webhook" to add it.

![Webhook added](images/WebHook%20added.png)

### Step 1.3.1: Verify the webhook worked

Open your gitbash window and type the following code inside your app directory:
`git checkout -b dev`
- This will switch the branch from main to dev.

Check this worked by pushing the "dev" branch to github repository.

![Pushing dev branch](images/Pushing%20dev%20branch.png)

Check the branches on github, it should show the dev branch inside the branches dropdown:

![Dev branch on github](images/Dev%20branch%20on%20github.png)

- The reason we created a dev branch is for the CI of CI/CD. This means that we can push changes to the dev branch which Jenkins then tests before it is CD into the main branch of code. 
- This is the whole idea of CI/CD.

## Step 2: Continuous Integration Merge Branch Implementation

So far you've set up and tested the CI part of CI/CD pipelines. What this means is with the current setup: 
- It listens for a push
- It then tests the push
- If it passes the test nothing happens right now as CDE isn't set up.

### Step 2.1: Setting up CI Merge Test Project

First, Navigate to the left side of your Jenkins dashboard, select "new item" and choose a name:
- e.g. name-project2-merge-test
- You are setting up a project that merges the code from the dev branch to the main code branch once it passes the test (automatically)

This time select the "copy from" as the project type. Select your project 1 from the CI setup as the copy.

![copy from project 1](images/copy%20from%20project%201.png)
- This will set up all the github connections for the new project as you did previously without having to do it all again manually. 
- It also copies the settings to which you will make some changes.

Next, navigate to job description and type something meaningful for this new project.

![project2 description](images/project2%20description.png)

Scroll to "Build Triggers" and deselect the "GitHub hook trigger for GITScm polling" that was selected for the previous job. 
- Deselecting the "GitHub hook trigger for GITScm polling" in the job configuration prevents Jenkins from automatically triggering builds when changes are detected in the GitHub repository.

Move to "build step" and remove the current one (which is a copy form the previous one)

Next, add a "post build action". Select both "Push only if build succeeds" and "Merge Results"
- Selecting "Push only if build succeeds" ensures that changes are only pushed to the repository if the build passes, preventing broken code from being deployed. "Merge Results" ensures the successful build's changes are merged back into the target branch, completing the merge process after a successful test.

![Git Publisher](images/Git%20Publisher.png)

Keep the rest as it is.

Scroll to the "branches" section, add "main" to the "Branch to push" and add "origin" to the "Target remote name"

![Branches for project2](images/Branches%20for%20project2.png)

**IMPORTANT BEFORE MOVING ON** - Ensure */dev is set in the SCM branch to build section:

![SCM branch to build project 2](images/SCM%20branch%20to%20build%20project%202.png)

-  By setting it up this way, you're ensuring that the second project (which handles the merge) is only triggered if the first project’s build is stable and successful. This means the merge process will only occur if the tests pass, preventing any unstable or broken code from being merged. Essentially, it creates a safeguard where the merge and subsequent actions only happen after a successful test, maintaining the integrity of your codebase.

### Step 3: Test Entire CI Pipeline

Make a change to the README.md file on your local machine.

Run `git status` command inside gitbash to check the change saved to the repo.

![git status for README test](images/git%20status%20for%20README%20test.png)

Run:

`git add .`
`git commit -a -m "Meaningful message"`

- This will add the changes ot the staging area and then ready them for pushing to the github repo. 

Now run:
`git push -u origin dev`

- This will trigger a push to the dev branch on github which will also trigger the test on jenkins. If it passes this test (project1) then project2 will begin and merge the changes form the DEV branch to the MAIN branch.
- If this happens then it passed the merge test and completed the CI/CD pipeline successfully.

![push to dev](images/push%20to%20dev.png)

Check Jenkins build history to see if both projects built successfully:

![Jenkins build proj1 and 2 history](image.png)


## Step 4: Continuous Deployment (CDE) Implementation & Test

### Step 4.1: Creating the CDE Test Project on Jenkins

To begin you need to set up a "new item" on jenkins and choose a name for the third project.

Select the "copy from" project type and copy the second project (merge project).

![Project2-merge-copy](images/Project2-merge-copy.png)

Add a meaningful description. 

![project3 description](images/project3%20description.png)

Keep everything the same and scroll to the SCM settings.

Inside the SCM setting change the "branch specifier" inside the "branches to build" to "*/main"

![Branches to build project 3](images/Branches%20to%20build%20project%203.png)

- Changing the "Branches to build" in the SCM settings of a Jenkins project tells Jenkins which branch to pull code from and build.
  - So if you set it to main, Jenkins will:
  - Pull code from the main branch.
  - Build and test only the main branch.

#### Launch EC2 "Production Environment" Instance

You now need to launch an EC2 instance that will serve as the production environment. Once the CI pipeline completes successfully in Project 3, Jenkins will deploy the tested code to this production EC2 instance.

Launch an instance using the app AMI. Using the same key pair and security group as usual.

Launch the instance. This is now the production environment that you will be able to push new code to by using the CI/CDE pipeline setup.

**--Move back to jenkins--**

Scroll to "Build Environment" and select "SSH Agent".
- This allows the agent to connect through SSH to the production environment (new AWS instance running the base code app)

Select "add" to add a new SSH key credential.

Next, select the "SSH Username with private key" as the kind of credential. Choose an ID, add a relevent description to differentiate the keys and add a username. 

![SSH credential for logging in to production env](image.png)

Next open a GitBash window on your local machine, navigate to the ".ssh" directory and run:
`cat <name of aws private key>` **Make sure to not expose this to anyone**

![cat aws private key for proj3](images/cat%20aws%20private%20key%20for%20proj3.png)

Copy the private key from your gitbah window to your jenkins text box.

Save it and select it from the dropdown menu.

![SSH agent complete for proj3](images/SSH%20agent%20complete%20for%20proj3.png)

Next, scroll to "Post-build Actions" and remove the current one as this is meant for project 2 (as this is a copy).

Now add a "build step" and select "Execute shell" from the dropdown options.

Input the following code into the execute shell textbox:

````
rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<EC2IPADDRESS>:/home/ubuntu

ssh -o "StrictHostKeyChecking=no" ubuntu@<EC2IPADDRESS> <<EOF
   cd app
   npm install
   pm2 kill
   pm2 start app.js
EOF
````
#### Explanation of what these commands do:

- rsync -avz: Syncs files in archive mode (-a), verbose (-v), and compressed (-z) for faster transfer.
- -e "ssh -o StrictHostKeyChecking=no": Uses SSH to connect and skips the host key verification prompt, which prevents SSH from asking to confirm the EC2 instance's identity. This is useful for automated scripts but slightly less secure.
- app: Local app folder to transfer.
- ubuntu@<EC2IPADDRESS>:/home/ubuntu: Remote user and path on the EC2 instance where files will be copied.

- ssh -o "StrictHostKeyChecking=no": SSH login without host key confirmation prompt (same reason as above).
- cd app: Enters the app directory.
- npm install: Installs Node.js dependencies listed in package.json.
- pm2 kill: Stops all current PM2 processes.
- pm2 start app.js: Starts the app using PM2 **with new code added to it**
- EOF: End Of File. This tells the agent to run multiple commands back to back. It basically runs a temp file that runs multiple commands and then deletes it (script).

Save the changes made to project 3 configuration. 

*Project3 Created*

#### Configure Project 2 to Trigger Project 3 on Successful Build

- As part of setting up **Project 3** (the CDE project), which is responsible for copying tested code to the production environment, you need to update the configuration of **Project 2**. 

- Specifically, configure Project 2 to trigger **Project 3** only if it runs successfully—i.e., after it has merged code from the `dev` branch to the `main` branch following a successful test. 

- This ensures that Project 3 only runs when the code is stable, allowing it to deploy the latest changes to the production EC2 instance.

Select project 2 and click "configure" on jenkins. 

Scroll to the "Post-build Actions", click add and choose "Build other projects" form the dropdown. 

Inside the "Projects to build" textbox select project 3 (your CDE project)

![Build other projects (3)](images/Build%20other%20projects%20(3).png)

- This tells project 2 to trigger project 3 (CDE project) ONLY if project 2 is successful.

Lastly, save the changes made.

*CI/CD Pipeline Now Ready*

### Step 4.2: Testing CI/CDE Pipiline

First, make a change to your app code. In my case I changed the "h2" main page header inside the "index.ejs" file in the app directory. 

![Change to h2 inside app code test](images/Change%20to%20h2%20inside%20app%20code%20test.png)

Move to your GitBash window on your local machine and run the following commands:
````
git add .

git commit -m "CDE test change"

git push -u origin dev

````

- This should trigger the CI/CDE pipeline, as the push to the `dev` branch activates the webhook configured in GitHub. Jenkins will detect the change and begin the pipeline process automatically.

The projects in Jenkins should be configured to run sequentially. Once a project (e.g., Project 1 or 2) completes successfully, it should automatically trigger the next project (e.g., Project 3) in the pipeline. This ensures a smooth flow from code testing to deployment without manual intervention.

![3 projects run successful](images/3%20projects%20run%20successful.png)

Check the applicaiton to verify the change to the header was made:

![alt text](image.png)

### Pipeline Execution Overview:

- After Project 1 successfully runs the tests, **Project 2** is triggered. This project is responsible for merging the code from the `dev` branch to the `main` branch, ensuring only tested and stable code is merged. Once Project 2 completes successfully, **Project 3** is automatically triggered, which deploys the changes to the production environment.