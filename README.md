# SE Demo Script

## Part 1 - Create local repo and connect to remote repo

### Create local Git Repo on you dev machine

Open the local folder you want your repo to exist in.
For this demo it will be the directory that you are going to deploy the Docker stack in and persist files.
For anyone not using containerization, it may just be the data directory of an Ignition install.

Create a file in that directory called .gitignore.
the .gitignore file will tell git what files in the directory to "ignore" and not version control.
For this demo, use the following 

Open a terminal at that directory.
Run the following command to create a git repo there:
```
git init .
```

*Note: Note where the git repo is being intialized. Customers just doing vcs on their config and projects will likely initialize this on the data directory. We are only doing this at this higher folder level because our entire demo config, projects, and deployment is what is being version controlled.

Create a remote remote repo in GitHub

Then run add the remote repo:
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

To view the localrepo as up to date run the command:
```
git status
```

Open the repo in Github to see your changes reflected there


## Part 3 - Best practices and guidance
### What files to version control?
Take an additive approach. Create the local git repo in a directory that contains the files you care about tracking, and as few other as possible. 
For example: If you only care about projects and not config, create your local repo on the data/projects directory.

### The .gitignore file
The ["Version and Source Control Guide"](https://docs.inductiveautomation.com/docs/8.3/tutorials/version-control-guide) from the Ignition User Manual provides a lot of guidance on version control with Ignition. One of the examples on that page is of a .gitignore file for a gateway tracking the entire data directory. The .gitignore file holds a list of the paths to items that you would like git to not track. For Ignition many of these files are going to be local-only files. Things like files that hold local memory tag values, logs, and certs. I've also added the ignition.conf file to my gitignore. The reasoning for this is to be able to provide different deployment modes in the ignition.conf file.

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

## Part 4 - Clone to Prod environment

Move to your production environment

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


## Part 5 - Changes in feature branches 
### Create a feature branch on the dev environment
Lets create a feature branch for some changes. We'll add a tag and a label to display it on a view.
Make sure you are up to date with the master branch first.
If not up to date with the origin then push or pull any changes.
Run the following command to create a feature branch and push it to the remote repo with the following commands:
```
git checkout -b featureTest
git push -u origin featureTest
```
featureTest is the name of our new branch. Go to GitHub to view new branch there.

### Add the new resources in designer and push to remote branch
Open a designer window to this gateway.
Add a new Motor tag structure from the Dairy Simulator. 
Create a new view and add a label to it.
Drag the amps from that motor's tags over to the label and save the designer.
Now checking the git status will show changes made to some files as well as some new files to track.
Run the Git add and commit again to commit all those changes locally.
Then run the following command to push those changes to the remote repo.
```
git push origin featureTest
```

### Pull request and merge in GitHub
Then check GitHub. You will see that there are new changes in the featureTest branch.
Review those changes and create a pull request.
It should find no conflicts and be allowed to merge. 
Select Merge pull request.
GitHub will tell you that the featureTest branch can now be safely deleted. Go ahead and delete the branch.

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

