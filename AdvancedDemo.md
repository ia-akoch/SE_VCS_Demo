# SE Demo Script - Advanced Git

## Part 1 - Containerized Deployments: Tips and Best Practices
### Persisting files to version control?
We've talked previously about only using version control on the files you care about. This could be projects, config, or both. With a containerized environment we have the choice of what files to persist to the host file system vs in a Docker named volume. What we prefer to version control can also help us decide what files we want to persist.
If you want both projects and config items and are using containerization fro your deployment, only persist the data/projects and data/config/resources directories to the host file system. Them create the local repo on a folder where both of those are persisted. You may even get into more infrastructure related items you would like to version control, like a Docker Compose file, or other environmental defining files.

Share Ignition service in docker-compose.yaml file. Show how volumes are configured.
```
      - ign-demo-vcs-modes-data:/usr/local/bin/ignition/data
      - ign-demo-vcs-modes-resources:/usr/local/bin/ignition/data/config/resources
      - ign-demo-vcs-modes-projects:/usr/local/bin/ignition/data/projects
```
ign-demo-vcs-modes-data:/usr/local/bin/ignition/data - This line persists the entire data directory in a Docker named volume. It will keep not only the entire config and projects, but other "local only" files that hold things like the UUID and certs that make this gateway an unique instance from the others using the same remotes repo.

ign-demo-vcs-modes-resources:/usr/local/bin/ignition/data/config/resources - This line persists out config/resources directory to the host file system, allowing that to be controlled in a local git repo. It contains all of the gateway configuration items across all deployment modes.

ign-demo-vcs-modes-projects:/usr/local/bin/ignition/data/projects - This line perists the projects directoy to the host file system, alloring that to be controlled in a local git repo. It contains all of the project resources on the gateway.


### Potential Permission issues when persisting both Docker named volumes and bind mounts
As discussed before, it can be beneficial to only track the directories you want with your vcs efforts. However, if you are using a deployment method like Docker and persisting just the projects and/or config directory you may notice that the gateway itself will lose all the local files that make it a unique deployment(Things like the UUID and certs) whenever you re-spin up that gateway. You can persist the entirety of the data directory in a Docker named volume alongside the specific directories that are persisted to the host file system. Doing this can cause potential permission issues, as the container user that is creating the volumes may not have access to create the needed files on the host system. To allow this to run you can start the container with user: 0:0 to run as root. We also recommend supplying the IGNITION_UID and IGNITION_GID for security reasons to run the Ignition service in the container as a specifically created application service account.


### .env file
the .env file holds environmental variables for the docker-compose file. This allows us to keep all these variables for determining how the gateway is spun up in one spot. It also keeps us form needing to edit the docker-compose file. There are 2 notable things about the .env file in this demo that can be good practice.
1. The .env file has been added to the .gitignore file. This keeps the different environment variables for separate environments out of the tracked files.

2. We do track a .env.example file, that provides anyone spinning up a new environment with our stack to have a template .env file to start out with.

Looking at the .env file we can see a new variable for deployment mode. This will allow us to pass the deployment mode through our docker image environment variable and not have to touch the ignition.conf file, since we aren't persisting that to the host anymore!


## Part 2 - Tags, Releases, and Github Actions
### What are they and why?
Whenever your repo hits a state that you want to promote to production, it can be a good time to create a release. A release is a distinct version of the repo at a certain point in time. They are created based on tags, which are added to commits. Because of this it can be a good idea to add a tag to any commit that you may want to push to a QA/test environment.


### Let's create a release!
Instead of manually typing git tag in a terminal, we can use a GitHub Action to standardize our release process. This ensures that every release follows the same versioning schema and automatically generates a changelog.

Open create-release.yml in Github under Actions>Create Release>create-release.yml.

- Trigger: Notice this uses workflow_dispatch. This means the release is intentional—a human (the SE or Dev) triggers it from the GitHub UI when they are ready to "freeze" a version of the code.
- Version Auto-generation: If the user leaves the version as auto, the script uses a shell command to generate a timestamp: yyyy.mm.dd.hh.ss. This ensures uniqueness without having to track "v1, v2, v3" manually during a fast-moving demo.
- Automated Changelog: The script uses gh release list and git log to find every commit made since the last release. It automatically formats these into a markdown file, so your release notes are always accurate to the code.
- GitHub CLI: We use the gh release create command. This not only tags the commit but creates a formal Release object in GitHub that our deployment scripts can then "look up."

Create new release by going back to Actions>Create Release and "Run Workflow", keep the 'auto' tag to automatically tag the release according to the .yml
1. selecting run workflow will tak you to a summary page where you can click on the action and view the process got executed.
2. Go to the Code tab at the top of the page to view the Releases section on the right side to view the release that just got created. Click on this release to see all the commits that were included in this release.

## Part 3 - Github Deployment Actions, Runners, and Environment Variables
Now that we have a versioned Release, we need to get it onto our Test and Production servers. We use two nearly identical workflows: deploy-to-test.yml and deploy-to-prod.yml.

Open create-release.yml in Github under Actions>Deploy to Test>deploy-to-test.yml and explain the flow:
1. Version Resolution: The first job (resolve-version) checks if you entered a specific tag or left it as latest. If latest, it queries the GitHub API to find the most recent release tag.
-- Side Note: you can also enter a specific release tag if you need to roll back to a previous version
2. Environment Targeting: Notice the environment: test (or prod) property. This is a critical GitHub feature. It allows us to define different secrets for different servers.
3. The "Self-Hosted" Runner: Note the runs-on: [self-hosted, test]. This indicates that the GitHub Action isn't running in the cloud; it is running on a small agent installed directly on our Ignition server. This allows GitHub to use a local service on the test and prod servers to execute these commands.

### Github Self-Hosted Runner
Once the runner starts on the Ignition server, it performs the heavy lifting of pulling the code down and refreshing the gateway config we used to do manually:
1. Permissions Management: It uses chown to temporarily take ownership of the folder to perform Git operations, then restores permissions to the Ignition user (2003) so the Gateway can read the files.
2. Pull down new changes in release: git reset --hard "$VERSION" is used. This is the cleanest way to deploy; it wipes any local changes and ensures the server exactly matches the Release tag.
3. Scan the Config and Projects API: Finally, we trigger a curl request to the gateway’s Scan Config and Scan Projects endpoint to pickup the new changes.

### Github Environment Variables
Now you may have wondered, "How does the config and project scan work without a token for the APIs?". That is an excellent question. For these scripts to work, GitHub needs to know where your servers are and how to talk to them. We manage this in the Settings > Environments section of the repository. You will notice we have 2 enviornments setup: one for test and prod. Inside of these environments we have Secrets and Variables:
- Variables are for non-sensitive data (like file paths). They are environment-specific, so REPO_PATH might be /home/se-admin/git-testing/test on Test but /home/se-admin/git-testing/prod on Prod.
- Secrets are for sensitive data (API tokens). Once entered, they are encrypted and cannot be viewed by anyone, only used by the automated scripts. 

## Summary
So in Summary, we use Github Actions to standardize our Releases and deployments to maintain consistency in our CICD pipeline:
1. Develop locally and push to main.
2. Release: Run "Create Release" to tag the code.
3. Deploy to Test: Run "Deploy to Test." GitHub connects to the server, pulls the specific tag, resets the files, and tells Ignition to refresh.
4. Deploy to Prod: Once Test is confirmed, run "Deploy to Prod" to move the exact same code to the live environment.
5. Next we will talk about how to create the automations with sefl hosted runners and github environements.