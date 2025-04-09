# Guide to setting Up Continuous Integration (CI) in Jenkins

## Step By Step Guide:

## Step 1: Test making projects on jenkins and chaining them together

### Step 1.1: Make Your First Project

Click "New item" on the left side of your jenkins dashboard.

Next, enter  a meaningful name for your first project (also known as job)

Ensure you select "Freesyle porject" as this is the most flexible option.

![alt text](/images/first%20project%20name.png)

Select next and add a meaningful description. 

Ensure you tick the "Discard old build" checkbox and set the "max # of builds to keep" to 3.

![alt text](/images/first%20project%20description.png)

The reason for this is:
- The idea is for the project to onyl keep the most recent three builds. 
- Specifically making sure to keep the last successful build and then the last 3 builds.
- Note:
  - It's not getting rid of compile files but more the build history and configuration used.

Now, skip to build steps.

Choose "execute shell" form the dropdown and write a test command. 

Note:
- This command will demonstrate the the project did indeed work or not.

![Execute shell for first project](/images/Execute%20shell%20for%20first%20project.png)

Select "build now" button to build the project 1.

![project 1 built](/images/project%201%20built.png)

The green tick indicates the build was successful.

Select the build (click the "#1") and select "console output".
- This will show what the console did and verify the "execute shell" you entered in the configuration of the project.
  - in my case `uname -a`

![Console output project1](/images/Console%20output%20project1.png)

*First project creation complete*

### Step 1.2: Make a second project to test again

First, select "new item" and choose a name for the second project.

Choose the same settings as earlier:
- Freestyle project type
- Check discard old builds
  - Max # of builds to keep
- Select "execute shell" but this time enter `date` as the command.

![Project test 2](/images/Project%20test%202.png)

Lastly, test it worked by selecting "build" on the left side of the project 2 dashboard.

![Project test 2 success](/images/Project%20test%202%20success.png)

If it displays the date in the "console output" after building is successful then you have verified it worked correctly.

### Step 1.3: Chaining the two test projects

**Goal:** If one project builds successfully then build the next project in line.

First step is to click inside the first test project you created earlier and select "configure" on the left side of you jenkins page.

Scroll to the "Build other projects" section.

![Build other projects test](/images/Build%20other%20projects%20test.png)

Ensure you select the "Trigger only is build is stable".
- The makes sure it only builds the second project if the first one builds successfully.

Save the changes and navigate back to your dashboard. 

#### Step 1.3.1: Test the chaining was successful

Click the build button on the first project test:

![Build to test chain](/images/Build%20to%20test%20chain.png)

This should trigger the first project to build and then start the second project once the first one has been built.

![Second project started](/images/Second%20project%20started.png)

*Testing jenkins complete*

## Step 2: Continuous Integration (CI) implementation

### Step 2.1: Link jenkins to github repo using SSH keys

### Step 2.1.1: Setup SSH public key on github

First you need to generate an SSH key pair for this repo that jenkins will use to connect with.
- This is good for security and very good for automation as we can give it to a tool like Jenkins and without our intervention it can unlock the resource without our manual input. 
- This is how it will go and get our source code form our repo. 

Open a gitbash window on your local machine and naviagte to the .ssh directory.

Next, type the following command to generate an SSH key pair:
`ssh-keygen -t rsa -b 4096 -C <"your email address">`
- Make sure you use your email address.

Choose a name for your key pair to differentiate them from previous keys.

![SSH key pair name](/images/SSH%20key%20pair%20name.png)

Skip the password (unless you want to use one).

Now `ls` to check if the key pair was created:

![Key pair created](/images/Key%20pair%20created.png)

Navigate to github select your repo's settings at the top of your repo page (not your github settings)

On the  left side of your settings page select "Deploy Keys"

Select "add new" and choose a name.

Open your gitbash on your local machine and run the following command to print your public key:
`cat <name of your key.pub>`

Then navigate back to github and paste that public key into the key text box:

![Public key](/images/Public%20key.png)

Ensure you check "Allow write access"

Finally select "add key".

![Key added to github](/images/Key%20added%20to%20github.png)


### Step 2.1.2: Setup private key on jenkins

Select "New item" on the left side of your jenkins dashboard and enter a name.

Select the freestyle project type as you did in the earlier test projects.

Choose a meaningful description for the project:

![CI project description](/images/CI%20project%20description.png)

Check the "Discard old builds" setting again and set the "max # of builds to keep" as 3.
- Again this means that the project will only keep the most recent three builds. 
- Specifically making sure to keep the last successful build and then the last 3 builds.
- Note:
  - It's not getting rid of compile files but more the build history and configuration used.

Check the "GitHub project" option as this project will be linked to github. 

Navigate to your github repo tab on google and copy the url. Paste this url in the "Project url" textbox:

![github url for project url](image.png)
![Project url textbox](/images/Project%20url%20textbox.png)

Next, scroll to the "Source Code Management" (SCM) and check the "git" option. 

Navigate back to your github repo and copy the SSH url from the "code" section.

![SSH github url](/images/SSH%20github%20url.png)

Paste that url inside the "repository URL" on jenkins:

![SCM repo url](/images/SCM%20repo%20url.png)

Now add the key in the "credentials" drop down menu

![SSH usernamer with private key credentials](/images/SSH%20usernamer%20with%20private%20key%20credentials.png)

Add a name, description and username to it:

![Name, description and username of priv key](/images/Name,%20description%20and%20username%20of%20priv%20key.png)

Ensure you navigate to your gitbash window on your local machine and run the following command inside your .ssh directory
`cat <name of private key>`

Copy the entire private key including: 
- "-----BEGIN OPENSSH PRIVATE KEY-----" and the:
- -----END OPENSSH PRIVATE KEY-----
- **DO NOT SHARE THIS KEY WITH ANYONE**.

Paste it into the private key textbox:

![Private key text box jenkins](/images/Private%20key%20text%20box%20jenkins.png)

Next, change the "Branch Specifier (blank for 'any')" to "*/main"

![Branch Specifier](/images/Branch%20Specifier.png)
- We change the branch specifier in Jenkins to match the exact branch we want to build (e.g., main) when using a private SSH key, ensuring Jenkins pulls the correct code from the private repo using the right authentication method.

Navigate to the "Build steps" and select "Execute shell". In here type the following command `ls`.
- This will test whether the connection between jenkins and your repo on github was successful as it will list the files in your GitHub repo in the "console output".

Save the changes and then select "build" to build the project you jsut created. 

Lastly, navigate to the built project once it shows as successfully built (green tick). 

Select "console output" and check the output. It should print the ls command and show the contents from your github repo. 
- In my case it should show the "app copy"

![Console output after jenkins and github connection check](/images/Console%20output%20after%20jenkins%20and%20github%20connection%20check.png)

*jenkins to github link success with SSH keys*

### Step 2.2: Adding NodeJs to jenkins

It is now necessary to install NodeJs on Jenkins.

Navigate to the CI project configuration settings and select the "Provide Node & npm bin/ folder to PATH" checkbox:

![nodejs checkbox](/images/nodejs%20checkbox.png)

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

### Step 2.3: Create a WEBHOOK

Navigate to the configure settings inside the CI project again.

Change the "Branch Specifier (blank for 'any')" from "*/main" to "*/dev"

![Branch specifier dev](/images/Branch%20specifier%20dev.png)

Scroll to the "Build Triggers" and select the "Github hook trigger for GITScm polling"

![BUILD TRIGGERS GITSCM](/images/BUILD%20TRIGGERS%20GITSCM.png)
- This tells github to listen for a webhook.

Next, move to your github tab on google.

Select "add webhook" 

![Webhook link github](/images/Webhook%20link%20github.png)

Copy your CI project url from jenkins tab and paste it into the "Payload URL" on github. 

Ensure you add "/github-webhook/" to the end of the url you paste into github.

![Payload url](/images/Payload%20url.png)

Leave the content type as it is and no secret.

Ensure you "Disable" SSL verification as this is only for test purposes. 
- You would enable this if you were in production and not testing.

Select "Just the push event" and select the "Active" checkbox.

![Disable SSL](/images/Disable%20SSL.png)

Select "add webhook" to add it.

![Webhook added](/images/WebHook%20added.png)

### Step 2.3.1: Verify the webhook worked

Open your gitbash window and type the following code inside your app directory:
`git checkout -b dev`
- This will switch the branch from main to dev.

Check this worked by pushing the "dev" branch to github repository.

![Pushing dev branch](/images/Pushing%20dev%20branch.png)

Check the branches on github, it should show the dev branch inside the branches dropdown:

![Dev branch on github](/images/Dev%20branch%20on%20github.png)

- The reason we created a dev branch is for the CI of CI/CD. This means that we can push changes to the dev branch which Jenkins then tests before it is CD into the main branch of code. 
- This is the whole idea of CI/CD.


