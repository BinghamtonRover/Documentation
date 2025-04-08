---
icon: github
---

# Using GitHub

## Cloning a repository

In most cases, the repository would have already been created for you and you just need to contribute to it. Your first step is to download (**clone**) the repository. Copy the URL of the repository and run the following

```bash
$ git clone URL folder-name
# Clones the repository at URL to the folder of your choice
$ cd folder-name
```

This creates a **remote repository** called `origin` , which holds the URL and up-to-date information about the repository on GitHub. To work on a feature, follow the standard git procedure:&#x20;

```bash
$ git switch -c my-feature 
# Write code
$ git commit -a -m "Your message"
# Repeat until the feature is ready for review
```

## Syncing remote and local branches

### Pushing&#x20;

When you're ready to share your code, just run \`git push\`. You'll probably get an error like the following:

```
fatal: The current branch temp has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin my-feature

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.
```

This means that you have a local branch called \`my-feature\`, but Git doesn't know which _remote_ branch on GitHub (`origin`) corresponds to this branch. This is usually because the local branch is new and there is no remote branch for it yet. You can create it by following its advice:

```bash
$ git push --set-upstream origin my-feature
# Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
# remote:
# remote: Create a pull request for 'my-feature' on GitHub by visiting:
# remote:      https://github.com/BinghamtonRover/Your-Repo/pull/new/my-feature
# remote:
# To https://github.com/BinghamtonRover/Your-Repo.git
#  * [new branch]      my-feature -> my-feature
# branch 'my-feature' set up to track 'origin/my-feature'
```

{% hint style="success" %}
You can use a shorthand here: `git push -u origin my-feature` . As the above error suggested, you can make sure Git does this automatically by running `git config --global push.autoSetupRemote true` .&#x20;
{% endhint %}

The output here shows you which URL it pushed the new branch to, and `my-feature -> my-feature` means that it pushed a local branch called `my-feature`to a remote branch called `my-feature`. It also says `[new branch]`, and that your branch was set up to "track" `origin/my-feature`.&#x20;

Pushing a branch pushes any commits that are not already on the remote, but only when you run the command. If you commit more changes after pushing — push again.

### Pulling

When you sit down to work, it's a good idea to check if there are any updates to the `main`branch:&#x20;

```bash
$ git switch main
$ git pull
```

You might see a message like `Already up to date` ,  which means no changes were made, or a message that mentions a fast-forward, which means someone pushed changes to the remote `origin/main`that are now synced to your local `main`. This applies to more than just the main branch though. If a team lead made changes to your feature branch, or another member was working on your branch, you'll need to pull that as well before you start working.&#x20;

{% hint style="warning" %}
**Stick to your own branch as much as possible**

As discussed in [working-with-branches.md](working-with-branches.md "mention"), committing to `main` will set you **ahead** of `origin/main`, meaning you won't be able to push or pull without conflicts. If you push to someone else's feature branch without coordinating, they might commit before pulling and be in the same position. In the ideal case, you would only push to your own branch and use a pull request (see below) to merge with the main branch.
{% endhint %}

To check whether you need to push or pull, you can always run git status:&#x20;

<pre class="language-bash"><code class="lang-bash">$ git status
# On branch temp
# Your branch is behind 'origin/temp' by 1 commit, and can be fast-forwarded.
#   (use "git pull" to update your local branch)
$ git status -sb 
<strong># ## temp...origin/temp [behind 1]
</strong></code></pre>

### Pruning

We use very short-lived feature branches: once a branch is merged, it is deleted and the canonical version of that code lives on in `main` . However, GitHub can only delete its own branches, meaning the remote branch will disappear but your local branch will live on. You can always **prune** the branches that have been deleted on the remote from your local repository:&#x20;

```bash
$ git remote prune origin
# Pruning origin
# URL: https://github.com/BinghamtonRover/YourRepository.git
#  * [pruned] origin/feature-branch
```

{% hint style="info" %}
This won't actually delete your local branches. Rather, it deletes the pointers your device has to the remote branches. You still have to delete the local branches with `git branch -D branch`. Using `-D`means it's okay to delete the branch even if it hasn't been merged to `main`. Even if its pull request was merged, Git has a hard time relating the two when we use squash merges.&#x20;
{% endhint %}

## Pull Requests

When you push a branch, it's publicly available for others to browse on GitHub or pull to their local repositories. When you're ready for a review, you open a **Pull Request**, which has a few benefits:&#x20;

* A custom title and descriptive message to represent your changes
* A conversation thread to discuss your implementation
* A commits page to view all the commits in your branch
* A page to view the final diff that will be applied to the `main`branch
* An opportunity for others to leave code reviews and highlight specific portions
* A chance for CI (continuous integration) tests to run and validate your code
* A big green button to press when you're ready to merge

To open a pull request, you can click on the link you got from `git push` , or go to the GitHub repository, click Pull Requests, click New Pull Request, then choose `main` on the left side and your feature branch on the right side.&#x20;

{% hint style="success" %}
If you aren't ready to merge your code but still want a review or draw attention to your branch, you can open a **Draft Pull Request**. Before pressing the green button to open your pull request. click on the dropdown arrow instead and choose draft. If you already created it, you can click `Convert to Draft` on the left.
{% endhint %}

## More Resources

* [GitHub's beginner tutorial](https://docs.github.com/en/get-started/start-your-journey/hello-world)
* [GitHub's Pull Request docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
* [reviewing-code-admins.md](../../writing-code/reviewing-code-admins.md "mention")
