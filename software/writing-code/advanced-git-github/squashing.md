---
description: What is "Squash and Merge" and why we use it
---

# Squashing

GitHub offers three ways to merge pull requests:&#x20;

* **Create a merge commit: A**dds all commits from the feature branch into the main branch, interleaving them with other commits in chronological order
* **Rebase and merge**: Takes all the commits and puts them on top of the latest commit on `main`
* **Squash and merge**: Rebases, then combines the commits into one big commit before merging.

The notable difference of squashing is that _it creates a new commit,_ one that doesn't correlate to any commit made by the original author, while also deleting the original commits of the author, along with all the important metadata that comes with them. That presents a very reasonable question: why? This document will attempt to explain why the benefits of squashing justify this seemingly odd behavior.&#x20;

## Non-linear history

Let's revisit that handy `gitlog`command from [#creating-a-new-branch](../../onboarding-course/git-course/working-with-branches.md#creating-a-new-branch "mention")

```bash
$ gitlog
# * (HEAD -> main) 096c148 Added a README
# * c5ac38c Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
# * e96ab34 Added Hello World
```

Notice how the commits are all in a straight line. It's very simple to reason about this history: first, we made a Hello World. Then we moved the greeting message, realized it generated a cache, ignored the cache, and then added a comment followed by a README. If any of these steps introduce a bug or new behavior, we can figure out exactly which one it was pretty easily. This is called a **linear history**.&#x20;

Let's see what a non-linear history looks like:&#x20;

```bash
# Add commits to a new branch, branch1
$ git switch -c branch1
$ echo This is file 1 > file1.txt
$ git add file1.txt
$ git commit -m "Added file 1"
$ git switch main

# Add commits to a new branch, branch2
$ git switch -c branch2
$ echo This is file 2 > file2.txt
$ git add file2.txt
$ git commit -m "Added file 2"
$ git switch main

# Merge the branches into main
git merge branch1 -m "Merge branch 1"
# Updating 096c148..f3399a7
# Fast-forward (no commit created; -m option ignored)
#  file1.txt | 1 +
#  1 file changed, 1 insertion(+)
#  create mode 100644 file1.txt
git merge branch2 -m "Merge branch 2"
# Merge made by the 'ort' strategy.
#  file2.txt | 1 +
#  1 file changed, 1 insertion(+)
#  create mode 100644 file2.txt
```

No merge conflict this time, but notice how the second merge was not a simple fast-forward. Why not? A fast forward only works when the branch you're merging already "knows" everything about your current branch (ie, it has all the target branch's commits). In this case, we're trying to merge `branch2`which never "found out" about the merging of `branch1`. Why doe this matter?&#x20;

```bash
$ gitlog
# *   968820b (HEAD -> main) Merge branch 2
# |\
# | * 730b7ad (branch2) Added file 2
# * | f3399a7 (branch1) Added file 1
# |/
# * 096c148 Added a README
# * c5ac38c Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
# * e96ab34 Added Hello World
```

Our history started off linear, until we started merging branches like this. Then you have two branches coming off of `Added a README`, which both merge back into one at `Merge branch 2`. Now it's not a straight line, so we call it a **non-linear history.** Due to the nature of trunk-based development, feature branches from `main`are going to appear all the time, and it won't take much for this history to grow.&#x20;

## Merging main into your feature branch

The above merge was a typical merge, which corresponds to the first option provided by GitHub: both branches were interleaved with each other before converging together back into one branch. What if we handled this complexity inside the branch itself? Let's try this again:&#x20;

```bash
$ git reset --hard 096c148  # go back to before the merge
$ git merge branch1 -m "Merge branch1 into main"
$ git switch branch2
$ git merge main -m "Merge main into branch2"
# Merge made by the 'ort' strategy.
$ git switch main
$ git merge branch2 -m "Merge branch2 into main"
# Fast-forward (no commit created; -m option ignored)
$ gitlog
# *   cd1243b (HEAD -> main, branch2) Merge main into branch2
# |\
# | * f3399a7 (branch1) Added file 1
# * | 730b7ad Added file 2
# |/
# * 096c148 Added a README
# * c5ac38c Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
# * e96ab34 Added Hello World
```

Was that better? Yes and no. We still have the non-linear history, but now the complex merge happened in the feature branch, and the real merge into main was a simple fast-forward

## Rebasing

At its core, the non-linear history arises because `branch2`started making its changes _before_ `branch1`merged into `main`. That problem is unavoidable, so we need a way to deal with it. Luckily, we can just pretend it never happened!&#x20;

```bash
$ git reset 096c148  # reset to before the merge
$ git merge branch1
# Fast-forward

$ git switch branch2
$ git reset --hard 730b7ad  # reset to before the merge
$ git rebase main  # NOT merge

$ git switch main
$ git merge branch2 
# Fast-forward

$ gitlog
# * 3670def (HEAD -> branch2) Added file 2
# * f3399a7 (main, branch1) Added file 1
# * 096c148 Added a README
# * c5ac38c Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
# * e96ab34 Added Hello World
```

With rebasing, we can pretend that `Added file 2` was written after `branch1`was merged into `main`, resulting in a linear history. Of course, this is not without its drawbacks:&#x20;

* Git has to rewrite every single commit on `branch2`to fake the history. That means this is not a viable option for commits that have already been pushed to a remote branch
* This doesn't magically avoid merge conflicts. In fact, it makes them worse, because conflicts have to be resolved while rewriting commits, meaning you might end up resolving the same commit again and again.
* While not specific to rebasing, many feature branches will ultimately contain annoying commits like "Working now" or "Fixed typo". These don't belong on the main branch.&#x20;

Squashing addresses these problems by waiting until the very end of a branch's life (ie, after reviews and tests have passed) to rebase and then combine all commits into one big commit before fast-forward merging.

* Since rebasing happens immediately before merging, the rebased commits aren't pushed
* Merge conflicts still have to be resolved, but they can be handled once per conflict
* All commits from the feature branch are now represented by a single, useful commit

This approach maintains a very useful guarantee of trunk-based development: **every commit on `main` is a known good commit that can be used in production.**&#x20;

## More Resources

* [The Git documentation on rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
* [This Stack Overflow answer about squashing](https://stackoverflow.com/questions/35703556/what-does-it-mean-to-squash-commits-in-git)
