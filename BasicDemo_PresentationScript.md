# Version Control with Ignition — Presentation Script

*A two-presenter demo script covering project and configuration version control on an Ignition gateway.*

**Cast:**
- **ADAM** — the "driver." Runs the terminal, the designer, and the gateway UI on screen.
- **REESE** — the "narrator." Explains concepts, sets context, and gives the audience the "why" behind each step.

**Setup before going on:**
- Latest Ignition installed on the demo machine
- Latest Git installed
- A GitHub account ready to log in to
- VS Code open with the Git Graph extension installed
- A terminal already open (as administrator on Windows) at the Ignition data directory

---

## Opening

**REESE:** Hi everyone, I'm Reese.

**ADAM:** And I'm Adam. Today we're going to walk you through version controlling an Ignition gateway end to end — projects, configuration, the whole data directory.

**REESE:** By the end of this session you'll have seen how to stand up a local repo on a dev gateway, push it to GitHub, deploy that same configuration to a prod environment, and use deployment modes to keep each environment's settings unique. We'll also do a feature branch and a pull request, just like you would in a real workflow.

**ADAM:** I'll start off driving — running the commands and clicking around in the designer. Reese is going to explain what we're doing and, more importantly, *why* we're doing it. Later we'll give him a chance to jump on the screen also and run our second environment.

---

## Part 1 — Create a local repo and connect it to a remote

### Creating the local Git repo

**REESE:** First thing we need is a Git repository on the dev machine. Adam is going to create that right inside the Ignition data directory. The reason we put it there — and not just on the projects folder — is so we can version control both the project files *and* the gateway configuration.

**ADAM:** Terminal's already open at the data directory. On Windows you'll want to open this as administrator. I'll run:

```
git init .
```

And there we go — we have a Git repo.

### What's in the local repo

**REESE:** Adam, can you show the `.git` directory?

**ADAM:** *(reveals hidden files in VS Code)* Here it is, you probably noticed it get created when I entered the command.

**REESE:** That's where Git stores everything about your local repository — the history, the branches, the staged changes. You generally don't touch it directly, but it's good to know it's there.

### The .gitignore file

**REESE:** Now, not everything in the data directory belongs in version control. There are local-only files — things like memory tag values, logs, certs — that are specific to *this* gateway and shouldn't be tracked. That's what `.gitignore` is for.

**ADAM:** The Ignition User Manual has a Version and Source Control Guide, and it includes a sample `.gitignore` for exactly this scenario — tracking the whole data directory. I'm going to create a `.gitignore` here and paste that in.

**REESE:** One small addition Adam is going to make — and you might want to do the same — is adding `ignition.conf` to the gitignore. We'll talk about why in Part 3 when we get to deployment modes. The short version: it lets each gateway run in a different deployment mode without fighting over that file.

**ADAM:** *(adds ignition.conf to .gitignore and saves)* Done.

### The README

**REESE:** While we're adding housekeeping files, let's add a README. Any repo benefits from a README — it gives the next person who opens this thing context on what it is and how to use it.

**ADAM:** I'll create a `README.md` and add a quick description of what this directory is, plus a note reminding anyone using it to set their deployment mode.

### A quick word on gateway backups

**REESE:** Before we go further, I want to address something that comes up a lot. Version control is great for tracking your configuration and projects — but it is *not* a full stateful backup of your gateway. Remember those files we just told Git to ignore? Logs, certs, local tag values — all the stuff that's unique to this running instance. None of that is in the repo.

**ADAM:** So you still want your gateway backups configured on a schedule.

**REESE:** Right. The way I'd put it: **Git is for CI/CD. Gateway backups are for disaster recovery.** Different tools for different jobs.

### Create the remote repo on GitHub

**ADAM:** Now let's set up the remote repo. I'm switching to the browser and logging in to GitHub.

**REESE:** Adam is going to click the green "New" button on the left, give the repo a name and an owner — usually yourself — and create it.

**ADAM:** *(creates the repo)* Repo created. I'll copy the URL.

### Connect local to remote

**ADAM:** Back in the terminal:

```
git remote add origin <Repo URL>
```

**REESE:** Heads up — if you've never set up Git on this machine before, that command might fail because Git doesn't know who you are. You'd run:

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```
Where "Your Name" is the username of your git user. and "your_email@example.com" is the email tied to your GitHub account. 
And then re-run the `remote add`.

### Stage and commit

**ADAM:** Now let's commit everything and push it up.
First we can see all the files we have to track with:
```
git status
```
Then to add and commit:

```
git add .
git commit -m "initial commit"
```

**REESE:** And that's Part 1. We have a local repo on the dev gateway, connected to a remote on GitHub, with an initial commit. Now let's actually make a change.

---

## Part 2 — Edit a resource and push the change

### Making a change in the designer

**ADAM:** I'm going to open the designer, create a new Perspective view, and drop a label on it with some text.

**REESE:** While Adam does that — what we're testing here is whether Git actually picks up changes made through the designer the way we'd expect. Spoiler: it does, because the designer is writing to the same data directory we're tracking.

**ADAM:** *(saves the view)* Saved. Back to the terminal:

```
git status
```

**REESE:** And there you can see the new files Git has detected. If you're in VS Code, this is a great time to look at the Source Control panel — you can see the diffs on individual files right there.

### Commit and push

**ADAM:**
```
git add .
git commit -m "Added Perspective view"
```

**REESE:** Let's hop into Git Graph to visualize the commit history.

**ADAM:** *(opens Git Graph)* You can see our two commits — the initial one and the new view. 

**REESE:** What about our remote repo?

**ADAM:** *(navigate back to GitHub)* Here you can see that the remote repo is still empty. Now I'll push to the remote:

```
git push -u origin master
```

Then check our status locally again

```
git status
```

Status is clean, local is up to date with the remote.

**REESE:** And if you flip over to GitHub, you'll see the changes reflected there too. That's one full edit-commit-push cycle.

---

## Part 3 — Deployment Modes

### What are deployment modes?

**REESE:** Okay, this is the part I'm most excited about. Deployment modes are a feature in Ignition that let you configure different "modes" for the gateway to run in. Each mode can override individual config items. When the gateway starts up, it checks which mode is active and applies that mode's overrides on top of the base configuration.

### Why we need them

**REESE:** Here's the problem deployment modes solve. When you share one Git repo across multiple environments — say dev and prod — you're declaring that the tracked files should be identical in both. But you might *want* to track a config file *and* have one or two values differ between environments. The classic example is the gateway name.

**ADAM:** Let me show that. *(navigates to the data/config/resources/core/system-properties/config.json file)* You can see right here that the system name lives in this config file. If we're tracking it, and dev and prod share a repo, they'd end up with the same name.

**REESE:** Which is exactly what we *don't* want. Deployment modes solve this — you create a mode per environment and override just the values that need to differ. Gateway name is the obvious one, but it's useful for anything that needs to be environment-specific.

### Creating a deployment mode

**ADAM:** Creating a mode is straightforward from the gateway UI. I'll navigate to **Platform > System > Modes**, click the blue **Create Mode +** button on the right, fill in the name, title, and description — I'll use `dev` for both name and title — and hit **Create Mode**. We can also make a `prod` mode while we're here to use later.

**REESE:** And just so you can see what that did under the hood, modes live on disk at `<IgnitionInstallDirectory>/data/config/resources`. We can see that we still have the core folder in this directory from before, but also have a dev and prod folder for those modes now. You could even create a mode by adding a folder there with its own `config-mode.json`.

### Creating an override

**ADAM:** Now I'll create an override for the gateway name. Navigate to **Platform > System > Gateway Settings**, click the three dots in the top right, and pick **+ Create Override**.

**REESE:** In that popup, select the mode you want the override to apply to. We'll use the `dev` mode we just made.

**ADAM:** *(creates override)* Now I can edit the System Name property to whatever this dev gateway should be called. Then I hit **Save Changes** in the top right.

**REESE:** Same as before, you can see this on disk under `<IgnitionInstallDirectory>/data/config/resources/<mode name>`. And you can author overrides directly in the file system too — just drop the files in and trigger a scan.

### Activating the mode

**ADAM:** Last step — telling the gateway *which* mode to run in. We do that in `ignition.conf`. I'm going to open it up and add this line under the Java Additional Parameters section:

```
wrapper.java.additional.<num>=-Dignition.config.mode=dev
```

Where `<num>` is the next number in the sequence.

**REESE:** And remember — this is exactly why we added `ignition.conf` to `.gitignore` earlier. Each environment sets its own mode in this file. If we tracked it, dev and prod would fight over the value every push.

**ADAM:** For the mode to actually take effect, we need to restart the gateway. I'll use `gwcmd`:

```
.\gwcmd.bat
```
This will make the gateway start up in our newly created dev deployment mode.

**REESE:** Once that is done restarting Adam will log back in to the gateway and we will see that the main gateway web page now shows the gateway running in teh dev deployment mode we created.

---

## Part 4 — Clone to a Prod environment

### Setting up the prod side

**REESE:** Now let's bring up a prod environment and pull the same configuration down.

**ADAM:** There are two paths here depending on where you're starting from. I'll explain both.

### Option A — Ignition is already installed

**REESE:** If Ignition is already installed and the data directory already exists, you'll init a repo and pull from origin. **Important caveat:** this will overwrite local files, so only do this on a fresh install where you don't have anything to lose.

**ADAM:** From the data directory:

```
git init .
git remote add origin <Repo URL>
git fetch origin
git reset --hard origin/main
```

### Option B — Fresh install, no data directory yet

**REESE:** If you don't have an environment at all yet, here's the cleaner path. Install Ignition but **uncheck "Start Ignition Now"** at the end of the installer. That way the data directory never gets created — and you can have Git clone it for you.

**ADAM:**
```
git clone <repo URL> <Local directory path>
```

**REESE:** Then update the deployment mode in the `ignition.conf` for this new environment — set it to `prod` or whatever you've named that mode — and you're ready to start the gateway.

### A change all the way through

**REESE:** Now let's see a change flow from dev to prod end to end.

**ADAM:** Back on the dev system, I'll open the gateway web page and add a programmable simulator device — I'll use the Dairy Sim program. Lets add and commit that change, then push it to the remote repo.

```
git status
git add .
git commit -m "added a programmable simulator device connection"
git push -u origin master
```

**REESE:** And on the prod environment I'm gong to pull that new change down with:

```
git pull origin master
```

**REESE:** Here's a wrinkle that catches people. We've updated the *file system* on prod, but the gateway's running config hasn't picked up the change yet. We need to trigger a file system scan. One way to do that is from the web UI: **Platform > Overview > Scan File System**

**ADAM:** The other is via the API at:

```
POST http://<gateway-URL>/data/api/v1/scan/config
```

**REESE:** Once the scan runs, check the devices page on prod — and there's the simulator. Change has made it all the way through.

---

## Part 5 — Changes in feature branches

### Creating a feature branch on dev

**REESE:** So far we've been working straight on master, which is fine for a demo but not likely how you'd do this in real life. Let's do it properly with a feature branch and a pull request.

**ADAM:** First I'll make sure dev is up to date with master — push or pull anything outstanding. Then:

```
git checkout -b feature/newFeature
git push -u origin feature/newFeature
```

**REESE:** The `feature/` prefix is a convention we're using for this demo — it tells anyone looking at the branch list that this is a feature branch, and the part after the slash describes what feature. If we hop over to GitHub, you can see the new branch there.

### Adding resources on the branch

**ADAM:** Back in designer. I'm going to add a new Motor tag structure from the Dairy Simulator, create a new view, drop a label on it, and drag the motor's amps tag over to bind it.

**REESE:** While Adam saves that — this is the kind of small, focused change you'd typically put in a feature branch. A new tag, a new view, one bound value.

**ADAM:** Saved. Let's see what changed, and get those all pushed to our remtoe repo:

```
git status
git add .
git commit -m "Added motor tag and view"
git push origin feature/newFeature
```

### Pull request and merge

**REESE:** Over in GitHub now. Adam will go to the **Pull requests** tab, click **New pull request**, pick the branches — `feature/newFeature` merging into `master` — and click **Create pull request**.

**ADAM:** *(creates PR)* And there it is. In a real workflow this is where your team reviews the diff. No conflicts here, so I'll click **Merge pull request**.

**REESE:** GitHub will then offer to delete the feature branch since it's been merged. Go ahead and do that — keeps your branch list clean.

### Pulling the merged changes to prod

**REESE:** Back on prod:

```
git pull origin master
```

**ADAM:** And this time we need to scan for *both* config and project changes, because this PR touched both.

**REESE:** Config scan, same as before — **Platform > Overview**, or:

```
POST http://<gateway-URL>/data/api/v1/scan/config
```

And projects — **Platform > Projects** in the UI, or:

```
POST http://<gateway-URL>/data/api/v1/scan/projects
```

**ADAM:** Open the project on prod and you should see the new view, the new tag, and the bound amps value live. Full feature, dev to prod, through a pull request.

---

## Closing

**REESE:** So that's the whole loop. A local repo in your data directory, a shared remote on GitHub, deployment modes to keep each environment's quirks isolated, and a real branching workflow for changes.

**ADAM:** A couple of things to remember on your way out: gateway backups are still your disaster recovery — Git doesn't replace them. And after any pull on a target environment, you need to trigger a file system scan for the gateway to actually pick up the changes.

**REESE:** The Version and Source Control Guide in the Ignition user manual has more detail on everything we covered, including the sample `.gitignore` Adam used. 

**ADAM and REESE:** Thanks for watching!
