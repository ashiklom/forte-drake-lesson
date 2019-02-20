---
title: Reproducible analysis of FoRTE canopy data using Drake
author: Alexey Shiklomanov
---

# Setup

Open RStudio.

Create a new project:
"Project -> New project"

Create the project from a GitHub repository.
"Create from version control -> Git"

The repository URL is `https://github.com/FoRTExperiment/FoRTE-canopy`.
Let's call the repository `forte-canopy-drake`.
(Also, pick a directory in which to store it).

Install the `drake` package.
"Packages -> Install -> `drake`"

# Exploring the data

Create a new R script called `drake.R`.
We won't do anything drake-related yet, but this is in anticipation of doing it later.
Since we'll be analyzing tabular data, let's make it easier with the `tidyverse`.

Let's load the `tidyverse` package and read in the A band list data.

```r
library(tidyverse)
a_bandlist_raw <- read_csv("data/A_bandlist.csv")
a_bandlist_raw
```

Another way to look at the data.

```r
glimpse(a_bandlist_raw)
```

Let's see what's actually varying in the data (i.e. what we want to summarize over).

```r
summarize_all(a_bandlist_raw, n_distinct)
```

Looks like it's all the same site, but several sub-plots, and multiple species.
Let's have a look at what some of these categorical variables actually are.

```r
count(a_bandlist_raw, Species)
count(a_bandlist_raw, Health_status)
count(a_bandlist_raw, Canopy_status)
```

A few things:
1. There's a problem in the `Species` column: `FAGR#` is probably supposed to be `FAGR`.
2. `Health_status` and `Canopy_status` have short codes that aren't very meaningful.

Let's fix these problems.
 
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

# Version control with git: Part 1

Click the "git" tab in RStudio.
There should be two files listed there:
`drake.R` and `forte-canopy-drake.Rproj`.

## Git theory -- briefly

A bit of Git vocabulary and related theory:

- A **repository** refers to a collection of files and their revision history in a directory that are tracked by Git. Here, our repository is the `forte-canopy-drake` directory.
- A **commit** is a snapshot of the repository (and all the files therein) at a particular point. A commit is usually accompanied by a helpful message describing what happened most recently.
- The **working directory** is the repository as you actually see and interact with it.

In the "Git" tab, click the "History" button to open a popup that shows a history of Git commits associated with the repository.
Click through some of these commits to get a sense of the history of how this repository evolved over time.

(**NOTE:** Git shows commits as "diffs" -- differences between files relative to the last commit. However, remember that under the hood, each commit stores the _full state of the repository_, not just the changes. Commits are rendered as "diffs" relative to the previous commit because when we look at the git commit history, or log, we are usually interested in the changes.)

## Changes in the working directory

Now, close the window and focus again on the open "Git" tab in the main RStudio window.
This window shows a graphical representation of the current "status" of our repository.
The "status" shows us the ways in which the **working directory** is different from the latest **commit**.
In this case, compared to the latest commit, we have added two new files: `drake.R` and `forte-canopy-drake.Rproj` (which was created automatically by RStudio).
The yellow `?` next to them indicates that they are new "untracked" files.

To demonstrate this further, let's make a change to one of the files that _is_ currently tracked by Git: `01_google_drive_test.R`.
Open this file, and make a trivial change -- for example, on line 3, change `jeff` to your name.
Save the file, and look at the "Git".
You should now see the file listed there with a blue `M` (for "modified") next to it.
Git has identified that the current state of this file is different from the one in the last commit.
How is it different?
Click on the file once in the "Git" menu to select it, then click "Diff" in the menu above.
This should open a window showing you exactly what change you made, along with a few lines before and after for context.

Notice the buttons "Stage chunk" and "Discard chunk" near the change.
"Chunk" refers to a set of changes grouped together by git based on their proximity.
Here, we are deleting -- red -- the line with "jeff" and adding instead -- green -- the line with your name, which could be interpreted as two separate changes, but git has conveniently grouped them for us.
"Stage" basically means "add to the list of things I'm about to commit" -- more on this in a bit.
"Discard" means delete the change and return the file back to the way it was.
Note that discarding changes is effectively **irreversible**.
However, here the change is minor and undesirable, so let's discard it.
Notice that once we do, the file should disappear from the "git" menu, and your name has changed back to "jeff" in the source code window.

## Your first commit: Adding the `drake.R` script

That was a change we didn't want to record.
However, we do want to record our new `drake.R` script.
In the "Review Changes" window, click on the `drake.R` file.
You should see the file displayed below, all in green (because it represents an addition).
Git is not tracking this file yet, and there is no way to introduce git to only part of a file.
To "introduce" Git to the file, we have to "stage" it.
You can do this with either the "Stage" button at the top of the window, or by checking the box next to the file.

When you do that, you should see that the status of the file has changed to an aquamarine `A` (for "added"), and the bottom panel has changed from "Unstaged" to "Staged".
The "Stage" is a temporary area that comprises changes that are about to be committed.
Consider a family photo:
If the commit is the photo, then the act of staging represents lining people up in front of the camera.
During a photo-shoot, you can move people in and out of the picture, modify their appearance, or take pictures of everyone at once, of everyone individually, or any combination thereof, depending on how you want your album to look.
The process of "staging" and "unstaging" works exactly the same way -- you can stage and commit a bunch of changes all at once, or do it selectively, depending on exactly how you want your commit history to look.

For now, let's just commit the entire `drake.R` file.
In the "Commit message" text box to the right of the file list, write a brief message describing what this commit represents, for example: "Create exploratory analysis in drake.R file", and click "Commit".
You should see a popup terminal window showing some progress.
Assuming there are no errors, you can safely close that.
You should now see that `drake.R` has disappeared from the status menu.
You should also see that a little info bubble at the top of the window says "Your branch is ahead of origin/master by 1 commit".
Ignore this for now -- we will talk about remotes (e.g. GitHub) later.
If you click on "History", you should see your commit was added at the top (end) of the commit history.

## Your second commit: Replacing the `Rproj` file

To solidify your understanding of creating git commits, let's try another one.
Notice that we have two RStudio project files in this directory -- the original `FoRTE-canopy.Rproj` and our new `forte-canopy-drake.Rproj`.
Let's replace the former with the latter.

In the RStudio file browser, delete the `FoRTE-canopy.Rproj` file.
When you do that, notice that it appears with a red `D` (for "deleted") in the git status menu.

In this exercise, we meant to do this.
However, it's worth pointing out that if you _didn't_ mean to do this, git makes it really easy to undo.
To un-delete the file, select it in the git status menu and then do one of the following:
- Click "Revert" at the top of the git menu.
- Click "Discard all" at the top of the "changes menu"

(Note that you can restore the file with e)

Now, click the boxes next to both the deleted `FoRTE-canopy.Rproj` and our `forte-canopy-drake.Rproj` to stage them.
Notice that the files were combined and a purple `R` (for "rename") appears beside them.
What happened?
Basically, git is being clever -- whenever you replace one file with another one that is sufficiently similar, git by default assumes you are renaming (or moving) the file rather than creating a new one.
Under the hood, this is because git tracks the overall _content_ of a repository, and the organization of that content is only a small part of that.
For more on this behavior (and some potential workaround), see [this StackOverflow answer](https://stackoverflow.com/a/15031787/2477097), but for our purposes, this is acceptable behavior, so we will move on from it.
To finish the commit, write an informative commit message in the box (perhaps, "Replace old Rproj file") and click the "Commit" button.
Confirm that the commit was stored by looking at the "History".

## Your third and fourth commit: Cleaning up the `drake.R` file

To belabor the point, let's practice with two more commits.
Here, the focus is on splitting a single batch of changes up into multiple commits.

Return to the `drake.R` script and remove all the intermediate lines that print to the console (e.g. `summarize_all`, `count`) -- after all, these were exploratory steps for developing the analysis that we no-longer need (and if we ever want to come back to them, they are now immortalized in the git history).
While we're at it, let's also tweak the plot to make it more aesthetically pleasing.
Specifically, let's clean up the Y-axis label, use a better theme, and rotate the X-axis labels.

```r
ggplot(a_bandlist) +
  aes(x = SubplotID, y = DBH_cm, fill = Species) +
  geom_boxplot() +
  ylab("DBH (cm)") +
  theme_minimal() +
  theme(
    axis.text.x = element_text(angle = 90)
  )
```

Now, save the file, and open it in the git "diff" window.
You should see that our changes were split up into two chunks:
In one, we delete the `summarize_all` and `count` lines, and in the second, we modify the plot.
There's nothing wrong with committing both of these changes together, but because these changes are thematically different, it may be better to split them into two different commits.

First, let's do the print statements.
Click the "Stage chunk" button corresponding to that section (or select the specific lines you want to stage -- holding "Shift" to select multiple lines -- and then click "Stage selection").
Once you do that, you should see that those lines have disappeared from the current view, and that in the status menu, `drake.R` now has two blue `M`'s next to it and has a blue square (rather than a check) in the box.
This is because that modifications to `drake.R` are now split -- some are in the working directory, and some are in the "staging area" (or "index"), the place where changes that are _about_ to be committed live.
(Note that this is only a git distinction -- the `drake.R` file currently on disk still has both files).
You can view the currently staged changes by switching the "Show" button from "Unstaged" to "Staged" -- that will now show only the removed print statements, and not the plot modifications.
When you are satisfied, commit the changes.

Now, let's repeat the process with the `plot` statements.
But, to further demonstrate the distinction between "staged" and "unstaged", let's add a step.
Once you have staged (but not committed) the plot changes, go back to the `drake.R` script and change the x-axis text rotation from 90 degrees to 45.
Save the file, then look again at the git unstaged area.
You should see that you have one unstaged modification, which you can either stage (overwriting the previous value of 90 degrees -- note that it if you do this, it's tricky to get back to the original value of 90), keep in the working directory but omit from the commit (in other words, the commit will have 90 degrees, but your working directory will still have 45), or discard (in which case your working directory will match what's in the staged area -- i.e. 90 degrees).

(_begin soapbox_)
One of the frequent complaints about the usability of git is the fact that it has a staging area, and that committing a change is a two-step process.
The point of this exercise was to show why I think the staging area is actually useful by giving you finer control over what goes into your commits and the ability to build commits incrementally.
The resulting better-organized commits make for a more useful history that is easier to revert, which in turn allows you to code with more confidence.
(_end soapbox_)

# Drake: Part 1

## Motivation

Consider our `drake.R` script.

Suppose we wanted to also fix the species codes so they matched the proper USDA NRCS Plants database codes:

```r
a_bandlist <- a_bandlist_raw %>%
  select(-X1) %>%
  mutate(
    Species = factor(Species) %>%
      fct_recode("FAGR" = "FAGR#",
                 "ACSA3" = "ACSA",
                 "POGR4" = "POGR"),
    Health_status = factor(Health_status, c("L", "M")) %>%
      fct_recode("alive" = "L", "dead" = "M"),
    Canopy_status = factor(Canopy_status, c("OD", "OS", "UN")) %>%
      fct_recode("canopy" = "OD", "subcanopy" = "OS", "understory" = "UN")
  )
```

In doing so, we have modified the `a_bandlist` data frame, so we now have to re-generate all the outputs that depend on `a_bandlist`.
For our simple analysis, that's just two things: The summary table and the plot.

But now let's extend the analysis.
(...maybe; might not need this)

Eventually, many analyses are sufficiently complex that it is impractical to keep track of all the inputs and outputs in a way that allows you to only re-run the code you need.
The best we can do is to just re-run all of the code from start to finish.
For small, fast tasks like ours, this is relatively painless.
But for more computationally-intensive tasks, this can quickly become at best mildly irritating and at worst a massive waste of time and resources.

Enter `drake`.

## Basic example

Let's take the summary script in our `drake.R` file and convert it to a `drake` workflow plan.

First, at the top of the script, load the `drake` package and a special configuration line (don't worry about what it does -- this is a `drake` package recommendation):
(...more steps...)

The end result will look like this:

```r
library(tidyverse)

library(drake)
pkgconfig::set_config("drake::strings_in_dots" = "literals")

plan <- drake_plan(
  a_bandlist_raw = read_csv(file_in("data/A_bandlist.csv")),
  a_bandlist = a_bandlist_raw %>%
    select(-X1) %>%
    mutate(
      Species = factor(Species) %>%
        fct_recode("FAGR" = "FAGR#",
                   "ACSA3" = "ACSA",
                   "POGR4" = "POGR"),
      Health_status = factor(Health_status, c("L", "M")) %>%
        fct_recode("alive" = "L", "dead" = "M"),
      Canopy_status = factor(Canopy_status, c("OD", "OS", "UN")) %>%
        fct_recode("canopy" = "OD", "subcanopy" = "OS", "understory" = "UN")
    ),
  species_summary = a_bandlist %>%
    group_by(Species) %>%
    summarize(
      mean_DBH = mean(DBH_cm, na.rm = TRUE),
      frac_alive = mean(Health_status == "alive", na.rm = TRUE),
      frac_canopy = mean(Canopy_status == "canopy", na.rm = TRUE),
      frac_subcanopy = mean(Canopy_status == "subcanopy", na.rm = TRUE)
    ),
  species_plot = ggplot(a_bandlist) +
    aes(x = SubplotID, y = DBH_cm, fill = Species) +
    geom_boxplot() +
    ylab("DBH (cm)") +
    theme_minimal() +
    theme(
      axis.text.x = element_text(angle = 90)
    )
)

dconfig <- drake_config(plan)
make(plan)
```

# Version control with git: Part 2

## Ignoring files

(To go after )

Often, we will have files in our repository that we don't want Git to track for various reasons.
Some examples include:
- Files containing passwords or other sensitive information
- Machine-specific configuration files
- Binaries from compiled code
- Cached outputs
- Large data files
