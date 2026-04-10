# SE Demo Script

## Before getting started
This demo script is meant to be used on an Ignition gateway deployed by any method to version control the Ignition data directory. THis will cover both project and configuration version control.

## Pre-requisits for those following along!
- Latest version of Ignition installed
- Latest version of Git installed
- Visual Studio Code with Git Graph extension installed
- GitHub account


## Part 1 - Create local repo and connect to remote repo

### Create local Git Repo on you dev machine

You will need to open a terminal or command promt to the data directory of you Ignition installation. (We will be using Visual Studio Code for this demo) Creating our git repo here will allow us to version control both project and config files.

Open a terminal at that directory.
Run the following command to create a git repo there:
```
git init .
```

Show .git directory

### The .gitignore file
The ["Version and Source Control Guide"](https://docs.inductiveautomation.com/docs/8.3/tutorials/version-control-guide) from the Ignition User Manual provides a lot of guidance on version control with Ignition. One of the examples on that page is of a .gitignore file for a gateway tracking the entire data directory. The .gitignore file holds a list of the paths to items that you would like git to not track. For Ignition many of these files are going to be local-only files. Things like files that hold local memory tag values, logs, and certs. I've also added the ignition.conf file to my gitignore. The reasoning for this is to be able to provide different deployment modes in the ignition.conf file. 

### Gateway Backups
Because of the files we are choosing to track and ignore, gateway backups are still recommended for disaster recovery. Version control in this use case is only providing us a tracking of our configuration and projects, not a full stateful backup of an Ignition gateway's instance. This is provided by a gateway backup, which should still be configured to have automated backups taken on a schedule in the gateway config.


### Create a remote remote repo in GitHub
Go to GitHub.com and Log in with your account
On the left there should be a green "new" button. Select that to create a new repo.
Give it a name, an owner.(likely yourself)
Copy the URL of that newly created repo to use locally.

### Connect local repo to your remote repo
Back in the local repo, run the following command to add the remote repo:
```
git remote add origin <Repo URL>
```

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

In the command line run the following commands to commit those resources and push to the remote repo:
```
git add .
git commit -m "Added Perspective view"
git push -u origin master
```

View changes in Git Graph

To view the localrepo as up to date run the command:
```
git status
```

Open the repo in Github to see your changes reflected there


## Part 3 - Clone to Prod environment

### Deployment modes for different environments
#### But why?
If you have the system-properties/config.json file tracked as part of your git repo you will notice that it holds the gateway name.
(Go to services/data/config/resources/core/ignition/system-properties and view the "systemName" property.)
Where this can cause issues is when the config directory is being tracked for config changes and you want gateways across different environments to use the same remote repo, but have different gateway names. A recommended solution for keeping gateway names unique to each gateway and not having that interfere with the controlled files is to use Ignition's deployment modes. Create an override for each of your environments and update the name for that environments respective mode. Deployment Modes can also be great practice for other settings that need to be unique across different evironments.

#### A look at how to create a deployment mode
Open the ignition.conf file and add the following line to the "Java Additional Parameters" section.
```
wrapper.java.additional.<num>=-Dignition.config.mode=dev
```
Where the num param is the next number in the sequence of params.

### Add local repo for Prod
####  Option A
Create a local repo and connect to the remote repo:
```
git init .
git add .
git commit -m "Initial commit"
git remote add origin <Repo URL>
git pull
```

#### Option B
If no environment exist yet, create the directory and clone the remote repo with the following command:
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


## Part 4 - Changes in feature branches 
### Create a feature branch on the dev environment
Lets create a feature branch for some changes. We'll add a tag and a label to display it on a view.
Make sure you are up to date with the master branch first.
If not up to date with the origin then push or pull any changes.
Run the following command to create a feature branch and push it to the remote repo with the following commands:
```
git checkout -b feature/newFeature
git push -u origin feature/newFeature
```
feature/newFeature is the name of our new branch, we used this naming convention to identify that branch as being a feature branch and what the name of that feature is. Go to GitHub to view new branch there.

### Add the new resources in designer and push to remote branch
Open a designer window to this gateway.
Add a new Motor tag structure from the Dairy Simulator. 
Create a new view and add a label to it.
Drag the amps from that motor's tags over to the label and save the designer.
Now checking the git status will show changes made to some files as well as some new files to track.
Run the Git add and commit again to commit all those changes locally.
Then run the following command to push those changes to the remote repo.
```
git push origin feature/newFeature
```

### Pull request and merge in GitHub
Then check GitHub. You will see that there are new changes in the featureTest branch.
Review those changes and create a pull request.
To make a pull request in GitHub select "Pull requests" at the top and select the green "New pull request" button.
Select the branch you want to merge and which one want it merged into, then select "Create pull request"
Then go to that pull request and select "merge"
It should find no conflicts and be allowed to merge. 
Select Merge pull request.
GitHub will tell you that the feature/newFeature branch can now be safely deleted. Go ahead and delete the branch.

### Pull those changes to the prod environment
In the prod environment run the following command to pull all the newly merged changes:
```
git pull origin master
```
Run the file system scan again. This time for both the config and project files:
- config
    - UI: 
        Platform>Overview
    - API: 
        ```
        http://<gateway-URL>/data/api/v1/scan/config
        ```

- projects
    - UI: 
        Platform>Projects
    - API: 
        ```
        http://<gateway-URL>/data/api/v1/scan/projects
        ```

Open the project on the prod environment to view changes.

## Part 5 - README file
It can be useful to have a README file for any repository. This provides documented context to what is in the repo, it's purpopse