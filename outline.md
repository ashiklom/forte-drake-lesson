---
title: Speaker notes
author: Alexey Shiklomanov
---

# Prerequisites

Software installed:

- R
- RStudio
- `tidyverse` R package
- Git (from git-scm.com or GitHub Desktop)

Create a (free) account on GitHub.

Make sure RStudio knows about Git:
"Tools -> Global options -> Git/SVN"

# Setup

## Git configuration

Main way to interact with git is through the command line.
To configure, in RStudio, go to "Terminal" tab (next to "Console") and type the following:

```
git config --global user.name Your Name
git config --global user.email your_email@email.com
```

Under the hood, this modifies the global git configuration, which generally lives in `~/.gitconfig`.

Most of the tutorial here will use the RStudio Git GUI, which has more limited functionality than the git CLI but is much easier to use.

## Create the project

Create a new RStudio project
New project -> Check "use git"

Make a data directory.
(In the console...)

```r
dir.create("data")
```

Download bandlist data.

```r
download.file("https://raw.githubusercontent.com/FoRTExperiment/FoRTE-canopy/master/data/A_bandlist.csv",
              "data/A_bandlist.csv")
```


# Committing changes

## First commit

Click the "git" tab in RStudio.
Should see 3 files: `.gitignore`, `data/`, and `forte-canopy-git.Rproj`.

Git will not track anything for you automatically.
_It is not Dropbox_.
Instead, think of Git as a wedding photo album.
- A **repository** refers to a collection of files and their revision history in a directory that are tracked by Git. Here, our repository is the `forte-canopy-git` directory. Think: photo album
- A **commit** is a snapshot of the repository (and all the files therein) at a particular point. A commit is usually accompanied by a helpful message describing what happened most recently. Think: Photo.
- The **working directory** is the files as you see them. Think: The stuff being photographed.

The menu you see shows the difference between your working directory and the last photo you took.
In this case, we have no photos, so everything shows up here...but git doesn't know what to do with these files yet -- photographer shows up, and these are the first people she sees, not sure what to do.

We want all of these files, so let's photograph them.
First, line up the people you want in this shot -- select the files.

- The **staging area** contains files as they are about to be committed. Think: Lining up a photo.

Then, click "Commit", add a message, and click "Commit".

View the history.
Note that the 

## Commit 2

Create a new script (`analysis.R`).

Read in the data:

```r
library(tidyverse)
a_bandlist_raw <- read_csv("data/A_bandlist.csv")
a_bandlist_raw
glimpse(a_bandlist_raw)
```

Now, commit it.

## Commit 3

Goal is to summarize.
What's varying in the data?

```r
summarize_all(a_bandlist_raw, n_distinct)
```

What do the categories look like?

```r
count(a_bandlist_raw, Species)
count(a_bandlist_raw, Health_status)
count(a_bandlist_raw, Canopy_status)
```

Look at Git.
These are modifications (hence "M").
Let's commit them.

## Commit 4

A few problems:
1. There's a problem in the `Species` column: `FAGR#` is probably supposed to be `FAGR`.
2. `Health_status` and `Canopy_status` have short codes that aren't very meaningful.

Let's fix them.
 
```r
a_bandlist <- a_bandlist_raw %>%
  select(-X1) %>%
  mutate(
    Species = factor(Species) %>%
      fct_recode("FAGR" = "FAGR#"),
    Health_status = factor(Health_status, c("L", "M")) %>%
      fct_recode("alive" = "L", "dead" = "M"),
    Canopy_status = factor(Canopy_status, c("OD", "OS", "UN")) %>%
      fct_recode("canopy" = "OD", "subcanopy" = "OS", "understory" = "UN")
  )
a_bandlist
```

And now, we can create our summary.

```r
species_summary <- a_bandlist %>%
  group_by(Species) %>%
  summarize(
    mean_DBH = mean(DBH_cm, na.rm = TRUE),
    frac_alive = mean(Health_status == "alive", na.rm = TRUE),
    frac_canopy = mean(Canopy_status == "canopy", na.rm = TRUE),
    frac_subcanopy = mean(Canopy_status == "subcanopy", na.rm = TRUE)
  )
species_summary
```

Let's also create a plot summary.

```r
ggplot(a_bandlist) +
  aes(x = SubplotID, y = DBH_cm, fill = Species) +
  geom_boxplot()
```

Commit these changes in several parts:
First, fixes to the data.
Second, the species summary.
Third, the plot.
While doing the plot, stage the change, then make several modifications (e.g. rotate X axis, fix X axis title), stage some and discard others.


# Remotes

A **remote** just another location where a repository is stored.
It can be another folder on the same computer, or a folder on a different computer.
GitHub, BitBucket, and GitLab (among others) are, at their core, just places where you can store a copy of your repository -- they just happen to provide some nice features for doing so (like browsing).

## Quick tour of GitHub

Start with Jeff's FoRTE-canopy repository, where this lesson's data comes from: https://github.com/FoRTExperiment/FoRTE-canopy

- Browse the directory structure
- Rendered `README.md` file
- Commits -- commit history.
    - Click on a commit to see the changes.
    - What does "updating scripts" mean?
- Look at specific code files (e.g. Google Drive test script)
    - "Raw" -- see the raw file; this is good for downloading files. Note the URL (this is how we downloaded the data)
    - "Blame" -- see which commits are responsible for which changes in the given file
    - "History" -- see that specific file's commit history
    
Now, let's look at an example of a massive collaborative project: RStudio -- https://github.com/rstudio/rstudio

- Look at commits (~25K!)
  - Click to see different author contributions
- Look at issues
  - Anyone can open an issue. In active projects, issues are often resolved super fast.
  - Issues can be bug reports, feature requests, or other questions -- often a great source of documentation; lots of workarounds. E.g. Issue #3843 (https://github.com/rstudio/rstudio/issues/3843)
  - Note the search keywords `is:issue is:open`. Remove `is:open` to view all issues.
- Look at pull requests
  - **Pull request** is a proposed set of changes.
  - GitHub facilitates discussion of changes, and line-by-line review of these changes.
  - I can take this, modify it, and adapt for my own use. If I want changes to be added to the original project, I can open a pull request.
  - Look at some examples of accepted ("merged") and rejected ("closed") PRs.
- Look at the "blame" on one of the files -- every change is not only attributable to someone, but also has an explanation.


## Create a GitHub repository

At top of screen, click "+" and "New repository".

Give the repository a name, and optionally, a brief description.

Let's keep the repository public.

(If you're worried that someone will steal your code, try searching GitHub for "forte" -- almost 2000 repositories!)

For now, _don't_ check "Initialize repository", and leave the other options as "None".

You now have an empty repository with instructions for what to do next.


## Upload your code to GitHub

Go back to RStudio.
There is no convenient way to register a remote via GUI, so have to do a bit more command line git.

## A bit of Git CLI

Go to the "Terminal".

The basic structure of git commands is as follows:

```
git <COMMAND> <OPTIONS>
```

To see a list of all `<COMMANDs>`, use command "help".

```
git help
# or...
git --help
```

To get help with a particular command, you can do:

```
git help <COMMAND>
# ...or...
git <COMMAND> --help

# E.g. for git config:
git help config
git config --help
```

## Creating a remote

Git commands related to remotes are under the `remote` command:

```
git remote --help
```

To create a new remote, use the `add` sub-command, which has the following syntax:

```
git remote add <NAME> <PATH>
```

Here, `<NAME>` can be anything.
A convention is to call your "main" remote (e.g. your personal GitHub) `origin`, so let's do that.
The `<PATH>` is the GitHub URL.
(Note that GitHub repositories have a standard format: https://github.com/username/repository).

```
git remote add origin https://github.com/ashiklom/forte-canopy-git
```

To see the list of all remotes:

```
git remote
```

This isn't very helpful, so let's also add the URLs by appending the `-v` (for "verbose"):

```
git remote -v
# ...or...
git remote --verbose
```

## Pushing local changes to a remote

The command to update a remote with your local changes is `push`, whose syntax is as follows:

```
git push <REMOTE> <BRANCH>
```

In this case, our remote is `origin`.
We will cover branches in the next section -- for now, all you need to know is that the default branch is called `master`.
Therefore, our command is:

```
git push origin master
```

You may be prompted for your GitHub username and password.
If this is successful, refresh the GitHub page of your repository and you should see everything.


## Pulling remote changes to your local repository

If you are working off a single machine, by yourself, you probably won't have to do this.
But, for completeness, let's step through it.

First, let's make a change to the code in the GitHub browser.
Specifically, let's use the browser to create a README.
Click the "Add a README" button, then create a short README file, then enter a commit message and commit it.
You should see the README appear, and be automatically rendered.

Now, return to RStudio.
Notice that nothing happened automatically.
Recall -- _Git is not Dropbox_; no synchronization ever happens unless you specifically tell it to.

The command to update your local repository from a remote is `pull`, and has similar syntax to `push`. 

```
git pull origin master
```

You should see a bunch of output describing what just happened, and you should see that the `README.md` file was added.


## Automatically tracking remote changes

We can save ourselves some typing, and enable some other useful features, by having our current branch (`master`) "track" a particular branch on the remote (again, don't worry about what branches are yet).
To enable this, let's repeat the `push` command, but with the additional `-u` (or the full version, `--set-upstream`) flag.

```
git push -u origin master
```

You should see a message like:

```
Branch 'master' set up to track remote branch 'master' from 'origin'
Everything up-to-date.
```

The first message means that your current work is now "tracking" what you have on GitHub.
For one, this means that if you just do `git push` or `git pull` without arguments, git will automatically push/pull from `origin`.
In addition, this means that you will get some useful information about the status of your local work relative to what's on GitHub.

To demonstrate, let's add one more commit to our local work.

Once you've done that, notice that the RStudio Git pane now shows the following message: "Your branch is ahead of 'origin/master' by 1 commit."
In other words, your local work is one commit ahead of what is currently on GitHub.
In addition, you can now use the "Pull" and "Push" buttons in the RStudio Git interface rather than relying on the command line.

Since we need to update our remote (GitHub), hit "Push".
You should see a popup showing the status of the operation.
Go back to GitHub and refresh.
You should now see that your latest commit has been added.

# Tags, branches, and HEAD

Now let's consider what that `master` word actually means.

In RStudio Git, open the "History" window and look at the list of commits.
Notice that the latest commit has several colored labels on it, in addition to the message: `HEAD -> refs/heads/master` and `origin/master`.
Notice also that in the far right, there is a column labeled "SHA" with a string of numbers and digits.

Each git commit is uniquely identifiable by a "SHA-1 hash" -- a 38-character string that is designed to have extremely low odds of duplication (1 in 2^80^, or roughly 1.2 x 10^24^).
What you're seeing here are the first 8 characters of that hash, which are usually enough to identify a commit within a single repository.
For instance, in the history menu, click on a commit and notice the "View file @ ..." link.

However, that hash is not very informative, so git also gives the option of labeling commits to make them easier to refer to.
Here, I go over 3 kinds of labels: tags, branches, and `HEAD`.
Combined with the raw commit SHA hash, these constitute the 4 kinds of git **refspecs**, which are used by many git commands.


## Tags

**Tags** are simple labels that apply to exactly one commit.
They are useful for marking more human-readable versions of programs, or more generally for other noteworthy parts of the history.
There is no way to manipulate tags in RStudio, so we will use the git CLI command `git tag`.
It has the following syntax:

```
git tag <TAG> <(REFERENCE)>
```

For example, to tag the last commit before the README as `pre-github`:

```
git tag pre-github <SHA>
```

Run that command, then refresh the "history" window and you should see it appear in yellow next to the commit in question.
Now, if you wanted to also tag this as "pre-readme", it would be easier to do because you can replace any occurrence of that commit's `<SHA>` with `pre-github`.

```
git tag pre-readme pre-github
```

A commit can have as many tags as you want.

**NOTE**: Tags are not pushed by default. To send tags to GitHub, you need to use the CLI command `git push --tags`.
Try it out, and note that once you do it, you can jump to specific "tags" on GitHub.


## Branches

**Branches** are, essentially, "dynamic" tags, meaning that they advance as you add commits.
They are best demonstrated by example.

Suppose we wanted to do our analysis in base R instead of using the `tidyverse`.
This is a significant change that will take several commits and non-trivial time investment.
Moreover, it's also a bit of a side-project; we probably also want to keep around the current version of our analysis and be able to switch back to it quickly.
This is a good candidate for branching.

### Create a branch

Click the "New branch" button (purple symbol) next to "master" in the RStudio git panel.
Keep "origin" selected as a Remote, and "Sync branch with remote" checked.
Note the output (and the commands used).

If successful, you will see that "master" has changed to "base-r".
Notice also that the Git History has changed a bit:
There is now a new tag "origin/base-r", the "HEAD" has changed to "refs/heads/base-r", and there is also the standalone label "master".


## Commit to the new branch

Now that we are on a new own branch, we can experiment with switching to base R.
In our first commit, let's change `readr`'s `read_csv` to base R's `read.csv`.
Commit the change, then look at the history.

Note that the green `HEAD` label has advanced by one commit, but `master`, `origin/master`, and `origin/base-r` are still glued to the previous commit.
This is exactly the behavior we want, because it makes it easy to jump back and forth between (or **checkout**) different branches.

To return to the previous version, checkout the `master` branch by clicking the "base-r" drop-down menu and selecting the "local" `master` branch.
You should now be back to using `read_csv`.

## Diverging histories

What makes branches _really_ cool is the ability to work on multiple "branches" of history simultaneously, as we'll demonstrate here.
On the `master` branch, change the plot color palette to `"Set2"` and commit it.
Now, view the "History" (you may have to refresh it to see the changes).
Notice that `HEAD` has now advanced one commit past `origin/master` and `origin/base-r`.

Where is the `base-r` branch?
By default, git only shows the history of the current branch and any branches that are its direct descendents.
To view all branches, click the "master" button and change it to "all branches".
That will show you aesthetically that your history has diverged after a common commit.

Let's do one more change on `base-r` to practice:
Checkout the branch (from the menu) and make another change.
Refresh and examine the "History".
Do you understand what happened?

## The `HEAD`

`HEAD` just refers to the tip of the current active branch.

## Updating branches on the remote

The remote's `master` and `base-r` branches are different from your current ones, which is why they have the additional `origin` label.
Use `git push` to update them.
Once you push `base-r`, you should see that `origin/base-r` has advanced to match up with `HEAD`.

## Merging

Use `git merge` to incorporate the changes of one branch into another.
