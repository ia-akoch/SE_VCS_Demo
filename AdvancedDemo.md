# SE Demo Script - Advanced Git

## Part 1 - Containerized Deployments: Tips and Best Practices
### Persisting files to version control?
We saw in the last video a work flow tracking the Ignition data directory. With a containerized environment we have must persist the files we want to track to the host file system as opposed to a Docker named volume.
If you want both projects and config items and are using containerization for your deployment, only persist the data/projects and data/config/resources directories to the host file system. Them create the local repo on a folder where both of those are persisted. You may even get into more infrastructure related items you would like to version control, like a Docker Compose file, or other environmental defining files.

Share Ignition service in docker-compose.yaml file. Show how volumes are configured.
```
      - ign-demo-vcs-modes-data:/usr/local/bin/ignition/data
      - ign-demo-vcs-modes-resources:/usr/local/bin/ignition/data/config/resources
      - ign-demo-vcs-modes-projects:/usr/local/bin/ignition/data/projects
```
ign-demo-vcs-modes-data:/usr/local/bin/ignition/data - This line persists the entire data directory in a Docker named volume. It will keep not only the entire config and projects, but other "local only" files that hold things like the UUID and certs that make this gateway an unique instance from the others using the same remote repo.

ign-demo-vcs-modes-resources:/usr/local/bin/ignition/data/config/resources - This line persists out config/resources directory to the host file system, allowing that to be controlled in a local git repo. It contains all of the gateway configuration items across all deployment modes.

ign-demo-vcs-modes-projects:/usr/local/bin/ignition/data/projects - This line perists the projects directoy to the host file system, alloring that to be controlled in a local git repo. It contains all of the project resources on the gateway.


### Potential Permission issues when persisting both Docker named volumes and bind mounts
As discussed before, it can be beneficial to only track the directories you want with your vcs efforts. However, if you are using a deployment method like Docker and persisting just the projects and/or config directory you may notice that the gateway itself will lose all the local files that make it a unique deployment(Things like the UUID and certs) whenever you re-spin up that gateway. You can persist the entirety of the data directory in a Docker named volume alongside the specific directories that are persisted to the host file system. Doing this can cause potential permission issues, as the container user that is creating the volumes may not have access to create the needed files on the host system. To allow this to run you can start the container with user: 0:0 to run as root. We also recommend supplying the IGNITION_UID and IGNITION_GID for security reasons to run the Ignition service in the container as a specifically created application service account.


### .env file
the .env file holds environmental variables for the docker-compose file. This allows us to keep all these variables for determining how the gateway is spun up in one spot. It also keeps us form needing to edit the docker-compose file. There are 2 notable things about the .env file in this demo that can be good practice.
1. The .env file has been added to the .gitignore file. This keeps the different environment variables for separate environments out of the tracked files.

2. We do track a .env.example file, that provides anyone spinning up a new environment with our stack to have a template .env file to start out with.

Looking at the .env file we can see a new variable for deployment mode. 
Open docker-compose file. and .env.example or .env file.
docker-compose file line: -Dignition.config.mode=${IGN_MODE:-dev}
.env file line: IGN_MODE=dev

This will allow us to pass the deployment mode through our docker image environment variable and not have to touch the ignition.conf file, since we aren't persisting that to the host anymore!

## Part 5 - Changes in feature branches 
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

## Part 2 - Tags, Releases
### What are they and why?
Whenever your repo hits a state that you want to promote to production, it can be a good time to create a release. A release is a distinct version of the repo at a certain point in time. They are created based on tags, which are added to commits. Because of this it can be a good idea to add a tag to any commit that you may want to push to a QA/test and/or Prod environment. A tag is basically a unique label added to your specific commit.

### Let's create a release!
To create a release in git hub you need a commit, and a tag associated with that commit. 

First thing we need to do is commit a change and create a tag for that commit.
- Let's add a Perspective view, and maybe drag a component on it.
- Save designer
- back in the terminal run a git add and commit
- Then run: 

You can tag on your commits, as tags are generally associated with commits. For our work flow I'm going to be tagging after a merge, as I want the release to be associated with a state of our main branch after a new feature has been merged.

We will navigate to GitHub.
- Select Releases on the right side
- Select "Draft a new release"
- In the Tag drop down we're going to select the "Create a new tag" button and name that tag whatever we want
- In the target dropdown we can either select the main branch, if we know it's at the state we want, or find our merge in the recent commits.
- Select the green "Publish release" button at the bottom

Now we have a new relese! This gives us a specific snapshot of what our tracked repo looked like at this time.

## Part 3 - Preview
### Automating actions
Instead of manually typing git tag in a terminal, we can use a GitHub Action to standardize our release process. This ensures that every release follows the same versioning schema and automatically generates a changelog. 

This is true for many actions other than releases as well. We'll cover automating actions in GitHub during our next installment.