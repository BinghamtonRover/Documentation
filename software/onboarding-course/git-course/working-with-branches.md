---
icon: code-branch
---

# Working with branches

Now that we know how to create commits and manage files, let's practice doing that on other branches, and merging back to `main` later. First, notice how `git status` has already been telling you at every step what branch you're on. We can also use `git branch`to list all the available branches and which one we're on.&#x20;

<pre class="language-bash"><code class="lang-bash">$ git status
# On branch main
# nothing to commit, working tree clean
$ git status -sb
<strong># ## main
</strong><strong>$ git branch 
</strong><strong># * main
</strong><strong>$ git branch -av
</strong><strong># * main c5ac38c Added a comment
</strong></code></pre>

{% hint style="success" %}
`git branch -av`will give `v`erbose information on `a`ll branches, including remote branches.
{% endhint %}

### Creating a new branch

Say we want to add a `README.md`file to our project. This file is shown to users when they look at your repository, so we'll want to take some time to get it right. Let's start by making a feature branch

```bash
$ git switch -c add-readme
$ git branch -av
# * add-readme c5ac38c Added a comment
#   main       c5ac38c Added a comment
```

{% hint style="success" %}
Good branch names are 1 to 3 words, separated by hyphens, in all lowercase.
{% endhint %}

Notice how even though we're on a new branch, both branches are actually at the same commit. Let's use another command to see the full history of the repository:&#x20;

<pre class="language-bash"><code class="lang-bash">$ git log --all --decorate --oneline --graph
# * c5ac38c (HEAD -> add-readme, main) Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
<strong># * e96ab34 Added Hello World
</strong></code></pre>

{% hint style="success" %}
From now on, we'll refer to that command as `gitlog`for short.
{% endhint %}

That's a long command, but worth saving or making an alias for, as it will show a lot more information in a much more concise way than the default `git log` will. We can see that `main` and `add-readme` both point to the same commit and `HEAD`points to `add-readme`. This makes sense: we've branched off but haven't done anything new yet. Let's do that.&#x20;

```bash
$ echo This is a new project > README.md
$ git status -sb
# ## add-readme
# ?? README.md
$ git add REDAME.md
$ git commit -m "Added a README"
# [add-readme 096c148] Added a README
#  1 file changed, 1 insertion(+)
#  create mode 100644 README.md
$ gitlog
# * 096c148 (HEAD -> add-readme) Added a README
# * c5ac38c (main) Added a comment
# * 92eca49 Added __pycache__/ to gitignore
# * f53d7d2 Moved the greeting message to data.py
# * e96ab34 Added Hello World
$ git branch -av 
# * add-readme 096c148 Added a README
#   main       c5ac38c Added a comment
```

Now we can clearly see that `main`ends at `Added a comment`while `add-readme`goes a bit further.

```bash
$ git switch main
$ git merge add-readme
# Updating c5ac38c..096c148
# Fast-forward
#  README.md | 1 +
#  1 file changed, 1 insertion(+)
#  create mode 100644 README.md
$ git branch --delete add-readme
# Deleted branch add-readme (was 096c148).
```

Because `add-readme`had everything `main`does, we only had to fast-forward to catch up to `add-readme`. This is the cleanest type of merge, but is still important to demonstrate.&#x20;

{% hint style="warning" %}
Our team exclusively uses GitHub pull requests to merge feature branches into `main` . If you try to merge branches this way and push your `main` , your push attempt will be blocked. This was mainly to demonstrate what GitHub is doing behind the scenes when you press "Merge" on the website.&#x20;
{% endhint %}

### Merge conflicts

We've already pointed out how merge conflicts can arise when two branches attempt to modify the same files. Let's take a look:&#x20;

```bash
$ git switch -c branch1
$ echo print("branch1") >> hello.py
$ git commit -a -m "Added print 1"
$ git switch main
```

So far, we've made a new feature branch and added a commit that modifies our file, but we haven't merged this branch yet. Now, let's branch off of `main`yet again and do the same thing:&#x20;

```bash
$ git switch -c branch2
$ echo print("branch2") >> hello.py
$ git commit -a -m "Added print 2"
$ git switch main
```

The setup is done:&#x20;

```bash
$ git branch -av
#   branch1 4f515a2 Added print 1
#   branch2 7456bca Added print 2
# * main    096c148 Added a README
```

Two branches, two commits, one file.&#x20;

```bash
$ git merge branch1
# Updating 096c148..4f515a2
# Fast-forward
#  hello.py | 1 +
#  1 file changed, 1 insertion(+)
$ git merge branch2
# Auto-merging hello.py
# CONFLICT (content): Merge conflict in hello.py
# Automatic merge failed; fix conflicts and then commit the result.
```

Git can't figure out what to do about `hello.py`. Let's see what it did:&#x20;

```python
import data

# Print a greeting to the user
print(data.GREETING)
<<<<<<< HEAD
print("branch1") 
=======
print("branch2") 
>>>>>>> branch2
```

Git sectioned off the difference changes, one section for `HEAD`(meaning, the current state of the `` main` `` branch) and the other for the incoming changes of `branch2`. As discussed in [merge-conflicts.md](merge-conflicts.md "mention"), the right thing to do is to analyze the situation with context. Let's assume for our example case that we really do want both print statements. The correct resolution is then:&#x20;

```python
import data

# Print a greeting to the user
print(data.GREETING)
print("branch1") 
print("branch2") 
```

Let's save and commit that

```bash
$ git commit -a
# [main 5fa56a5] Merge branch 'branch2'
$ git branch --delete branch1
# Deleted branch branch1 (was 4f515a2).
$ git branch --delete branch2
# Deleted branch branch2 (was 7456bca).
```

Chances are, your IDE will offer to help out here. For example, VS Code will open conflicting files in its Merge Editor, while Sublime Text offers a standalone application called Sublime Merge. Apps like these tend to have a three-paned layout: the current branch, the incoming changes, and the new resulting file.&#x20;
