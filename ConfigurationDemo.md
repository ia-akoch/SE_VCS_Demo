# SE Demo Script - GitHub Configuration

## Overview
This demo picks up where the Advanced demo left off. We now have workflows in place to create releases and deploy to test and prod environments — but before those workflows can run, GitHub needs to know about our servers. This demo covers the two steps required to make that happen:
1. Install a self-hosted runner on each server so GitHub Actions can reach them
2. Create the GitHub environments and configure their variables and secrets


## Part 1 - Self-Hosted Runners

### What is a self-hosted runner?
A self-hosted runner is a small background service that you install on your own server. It checks in with GitHub continuously and waits for work. When a workflow job targets `[self-hosted, test]` or `[self-hosted, prod]`, GitHub routes that job to the runner with that matching label — meaning the job executes directly on your Ignition server, with direct access to the local file system and network.

This is what makes it possible for our deployment workflow to do a `git reset` on the server and then hit the local Ignition API — none of that would be reachable from a cloud runner.

### Install the runner on the test server
Each environment needs its own runner. Start with the test server.

1. In your GitHub repository, go to **Settings > Actions > Runners**
2. Click **New self-hosted runner**
3. Select the operating system of your test server (Linux is used in this example)
4. GitHub will generate a set of commands specific to your repo based on the OS selected. Run those commands on the server to Download the runner package
- Note: these are the questions you will be asked:

5. Now go through the generated commands for the Configure section. Here are the settings that are used for this example:
-- "Enter the name of the runner group to add this runner to: [press Enter for Defualt]" This example will use the default
-- "Enter the name of the runner:" This example will use 'test' and 'prod' respectively
-- "Enter any additional labels:" This example will not enter any additional
-- "Enter name of work folder: [press Enter for _work]" This example will use the default '_work' folder
6. If done correctly, back in GitHub under **Settings > Actions > Runners**, you should see the runner appear as **Idle** within a few seconds

### Install the runner on the prod server
Repeat the exact same steps on the prod server, but use `prod` as the additional label during configuration:

After both runners are installed you will have two runners listed under Settings > Actions > Runners — one named `test` and one named `prod`.

### A note on runner permissions
The deployment workflow uses `sudo chown` and `sudo chmod` to temporarily take ownership of the repository path for git operations. The account that the runner service will need passwordless sudo access for those specific commands. If you see permission errors during deployment, that is the first place to check.


## Part 2 - GitHub Environments

### What are GitHub Environments?
GitHub Environments let you define environment-specific configuration that your workflows can reference. Instead of hardcoding server paths or tokens into your workflow files, each environment carries its own variables and secrets. The deploy-to-test.yml workflow targets the `test` environment and the deploy-to-prod.yml workflow targets the `prod` environment — so each automatically picks up the right values for that server.

There are two types of configuration stored in an environment:
- **Variables** — non-sensitive values like file paths and port numbers, visible in the UI
- **Secrets** — sensitive values like API tokens, encrypted at rest and never readable once saved

### Create the test environment
1. In your GitHub repository, go to **Settings > Environments**
2. Click **New environment**
3. Name it `test` (must match exactly what is in deploy-to-test.yml) and click **Configure environment**

### Add variables to test
Under the **Environment variables** section, click **Add environment variable** for each of the following:

| Name | Value |
|---|---|
| `REPO_PATH` | Absolute path to the local repository on the  server (e.g. `/path/to/local/repo`) |
| `GATEWAY_PORT` | Port the  Ignition gateway is listening on (e.g. `8088`) |

### Add the secret to test
Under the **Environment secrets** section, click **Add secret**:

| Name | Value |
|---|---|
| `IGNITION_API_TOKEN` | API token from the Ignition gateway |

To find or generate an Ignition API token, start the Docker stack which includes the Ignition gateway, go to the gateway webpages, and navigate to **Platform > Security > API Keys**. Keep in mind any security roles that you may want to incorporate with this API key.
https://docs.inductiveautomation.com/docs/8.3/platform/security/api-keys

### Create the prod environment
Repeat the same steps to create a second environment named `prod`:
1. **Settings > Environments > New environment**, name it `prod`
2. Add the same two variables with values pointing to the prod server
3. Add the `IGNITION_API_TOKEN` secret using the prod gateway's token

## Summary
The one-time configuration required to support automated deployments is:

1. **Runners** — Installed as system services on each server, labeled `test` and `prod`
2. **Environments** — Created in GitHub with `REPO_PATH`, `GATEWAY_PORT`, and `IGNITION_API_TOKEN` configured for each

After this configuration is complete, the full deployment pipeline from the Advanced demo is operational:
- Develop locally → push to main
- Run **Create Release** to tag a version
- Run **Deploy to Test** to push that version to the test server
- Run **Deploy to Prod** once confirmed on test