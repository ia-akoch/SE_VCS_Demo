# SE Demo Script

## Before getting started
This demo script is meant to be used on an Ignition gateway deployed by any method to version control the Ignition data directory. This will cover both project and configuration version control.

## Pre-requisits for those following along!
Need:
- Latest version of Ignition installed
- Latest version of Git installed
- GitHub account

Prefferred:
- Visual Studio Code with Git Graph extension installed


## Part 1 - Create local repo and connect to remote repo

First Ignition will need to be installed! If that has not been done yet, run the full Ignition install(or spin up a container) and your machine now!

### Create local Git Repo on your dev machine

You will need to open a terminal or command promt to the data directory of you Ignition installation. If running Windows it will need to be opened as administrator. (We will be using Visual Studio Code for this demo) Creating our git repo here will allow us to version control both project and config files.

Open a terminal at that directory.
Run the following command to create a git repo there:
```
git init .
```

### About the local Repo
#### the .git directory
Show .git directory.
This is where all of ther information about our local git directory is stored.

#### The .gitignore file
The ["Version and Source Control Guide"](https://docs.inductiveautomation.com/docs/8.3/tutorials/version-control-guide) from the Ignition User Manual provides a lot of guidance on version control with Ignition. One of the examples on that page is of a .gitignore file for a gateway tracking the entire data directory. The .gitignore file holds a list of the paths to items that you would like git to not track. For Ignition many of these files are going to be local-only files. Things like files that hold local memory tag values, logs, and certs. 

Lets create a .gitignore file in our directory and copy the contents in from the Version and Source Control Guide. I'm also going to add the ignition.conf file to my gitignore. The reasoning for this is to be able to provide different deployment modes in the ignition.conf file. We'll cover deployment modes later also.

#### README file
It can be useful to have a README file for any repository. This provides documented context to what is in the repo, so that any new user jumping in to look at that repo can read it and understand, at least at a high level, the purpose of that repo and how it is meant to be used.

Let's create a README.md file and add a quick description about what this directory is. You can also add notes to help people use the directory. For our instance let's add a note so people know to go in and set the deployment mode.

### Gateway Backups
Version control systems are great at providing us a tracking of our configuration and projects. For Ignition though, that is not a full stateful backup of an Ignition gateway's instance. As mentioned before, we are ignoring many local files that are unique to that deployed gateway. That type of full gateway instance snapshot is provided by a gateway backup, which should still be configured to have automated backups taken on a schedule in the gateway config.

Git is for CI/CD pipelines, and Gateway Backups are for disaster recovery.


### Create a remote repo in GitHub

***
For ICC Workshop:
Make sure both users in a group have GitHub accounts
Have one user create the repo and add the other user as a collaborator
***

1. Go to GitHub.com and Log in with your account
2. On the left there should be a green "new" button. Select that to create a new repo.
3. Give it a name, and an owner.(likely yourself) !For ICC workshop - make sure repo is public
4. Select create repo at the botton
5. Copy the URL of that newly created repo to use locally.

### Connect local repo to your remote repo

Back in the local repo, run the following command to add the remote repo:
```
git remote add origin <Repo URL>
```
This may fail if you have not set up a user and email for your git environment. To apply a user to the entire git environment on that machine run:
```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```
Where "Your Name" is the username of your git user. and "your_email@example.com" is the email tied to your GitHub account.
Now you can run the remote add again if needed.

### Add to staging and commit to remote repo

Next add the locally tracked files and commit them to the local repo with the following commands:
```
git add .
git commit -m "initial commit"
```


## Part 2 - Edit a resource and push changes

### Edit resource

Create a resource by:
- Start Ignition gateway (if not started already)
- Open designer
- Create a Perspective view and add a label with whatever text you want in it

In the command line run the following command to view changed files:
```
git status
```
*Note: If using VScode, this is a good time to jump back into that window and show the diffs for the actual files in the source control extension

### Commit resources and push to remote repo

In the command line run the following commands to commit those resources:
```
git add .
git commit -m "Added Perspective view"
```

View changes in Git Graph

Now push to the remote repo with:
```
git push -u origin master
```

To view the local repo as up to date run the command:
```
git status
```

Open the repo in Github to see your changes reflected there.


## Part 3 - Deployment Modes

### Deployment modes for different environments
#### What are deployment modes?
Deployment modes in Ignition allow you to configure a number of different "modes" for the gateway to run in. Each mode can have it's own unique override for individual configuration items. On start up the Ignition gateway checks which mode is it's "active" mode to start in, and uses the overrides for that mode as it's working configuration.

#### But why?
When we use a common git repo on a platform like GitHub to track multiple different local repos we are saying that all of those files are the same across each environment, with the exception of ignored files of course. But we may not want to ingore those files, because we do care about tracking that configuration item. This is where deployment modes come in. We can create separate modes for each environment, with configuration overrides that only impact gateways running in that mode. One vital example is with the gateway name.
If you have the system-properties/config.json file tracked as part of your git repo you will notice that it holds the gateway name.
(Go to services/data/config/resources/core/ignition/system-properties and view the "systemName" property.)
Where this can cause issues is when the config directory is being tracked for config changes and you want gateways across different environments to use the same remote repo, but have different gateway names. A recommended solution for keeping gateway names unique to each gateway and not having that interfere with the controlled files is to use Ignition's deployment modes. Create an override for each of your environments and update the name for that environments respective mode. Deployment Modes can also be great practice for other settings that need to be unique across different evironments.

### How to create a deployment mode
Modes can be created in the gateway UI by: 
- navigating to Platform>System>Modes
- Select blue "Create Mode +" button on right side of screen
- Enter the name, title, and description fields (enter the name and title as dev, and description if desired)
- Select "Create Mode" blue button

We can see see where our modes exist in the file system at: 
<IgnitionInstallDirectory>/data/config/resources
We could even create a new mode by adding a folder to this location with it's own config-mode.json if we wanted to.

### How to create an override
You can now create overrides specific to your deployment modes! The easiest way to do this is from the gateway UI. Simply navigate to the page you want to create an override for. We'll use our example for earlier of the gateway name. 

Let's create one for the gateway name example discussed earlier.
Navigate to: 
Platform>System>Gateway Settings

Click the 3 dots on the top right of the screen, and select "+ Create Override"

In the popup select the desired mode from the "Collection" dropdown and select the blue "Create Override" button
Let's do the dev mode for this example.

Now we can edit the System Name property to what our dev gateway should be.

When done making changes, select the blue "Save Changes" button in the top right.

Just as before, we can look at our new file in the file system at:
<IgnitionInstallDirectory>/data/config/resources/<mode name>
Again, we can create overrides from the file system too. This is done by moving files to these locations in the file system to make changes, and scan the file system to implement those changes to the gateway.

### How to set the deployment mode
Open the ignition.conf file and add the following line to the "Java Additional Parameters" section.
```
wrapper.java.additional.<num>=-Dignition.config.mode=dev
```
Where the num param is the next number in the sequence of params.

In order for this to take affect, the gateway must be restarted so that it can read from the mode form the ignition.conf file. You can do that by restarting the service or with the gwcmd restart.
*in VScode - .\gwcmd.bat*

## Part 4 - Clone to Prod environment

### Ignition install

### Add local repo for Prod
####  Option A - Ignition is already installed and data directory is created
Create a local repo and connect to the remote repo. We only want to do this if this is a new installation, as it will override everything locally.
Once again navigate to the data directory and type the following commands:
```
git init .
git remote add origin <Repo URL>
git fetch origin
git reset --hard origin/main
```

#### Option B
If no environment exist yet, install Ignition now but we will not let the service start so that the data directory does not get created. This can be done by unchecking the "Start Ignition Now" box when the installer is finished. This is also detailed in our [Version and Source Control Guide](https://docs.inductiveautomation.com/docs/8.3/tutorials/version-control-guide#installation-b). This will allow us to create the data directory files by cloning those from our remote repo.

Create the directory and clone the remote repo with the following command:
```
git clone <repo URL> <Local directory path>
```
Update env file and to match new environment

### Let's see a change all the way through!
#### Add a change to the local dev environment
Go back to the dev system and open the gateway web page.
This time let's add a programmable simulator device that users the Dairy Sim program.
Run the following commands to see what file has changed from our change, and to add and commit it to the local repo:
```
git status
git add .
git commit -m "added a programmable simulator device connection"
```

#### Push the new change to the remote repo
Let's push to the remote repo again with the following commands:
```
git push -u origin master
```
Now you can go to the remote repo and view the changes there if desired

#### Pull the new change to our prod environment
Now back on the prod environment run the following command:
```
git pull origin master
```

Because the file system has been updated and not the running configuration we will need to trigger a file system scan. This will cause our changes to take effect on the actual running gateway config. There are 2 standard methods to run a file system scan:
1. From the gateway UI. Navigate to Platform>Overview and select the "Scan File System" button in the top right.
2. From the gateway API. send a Post to the endpoint: http://<gateway-URL>/data/api/v1/scan/config

Once the scan is done running you can check the devices again and see your programmable device simulator there.
