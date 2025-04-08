---
icon: code-commit
---

# Making your first commit

With the explanation and background context covered in the previous pages, these next few sections will walk you through working with a Git repository.&#x20;

First, make sure you have git. You can do this by running `git --version`in your terminal. If you get an error message, you don't have it, or it's not set up properly and you should get it from [here](https://git-scm.com/downloads). As with any tool you install, don't forget to add it to your `PATH`.

To start, make a new directory and run `git init`inside of it

<pre class="language-bash"><code class="lang-bash">$ mkdir repo
$ cd repo
<strong>$ git init
</strong></code></pre>

Next, let's add a simple Hello World in Python

{% code title="hello.py" %}
```python
print("Hello, World!")
```
{% endcode %}

{% hint style="info" %}
Git strongly prefers that your files end with a blank new line. The code samples here won't show them, but in your IDE, be sure to do so, Otherwise, you might see subtle complaints from Git in various places.
{% endhint %}

Now let's see what Git has to say

```bash
$ git status
# On branch master
# 
# No commits yet
# Untracked files: 
#   (use "git add <file>..." to include in what will be committed)
#     hello.py
# 
# nothing added to commit but untracked files present (use "git add" to track) 
```

Let's break this down:&#x20;

> On branch `master`

Git will always start by reminding you what branch you're on. In this case, that's `master`, but convention is actually to use `main` . Let's create that:&#x20;

```bash
$ git switch --create main
# Switched to a new branch 'main'
```

Moving on:&#x20;

> nothing added to commit but untracked files present (use `git add`to track)

Git won't touch any files it doesn't already know about (called **tracked files**), so let's add our new file:&#x20;

<pre class="language-bash"><code class="lang-bash">$ git add hello.py
$ git status
<strong># On branch main
</strong># 
# No commits yet
#
# Changes to be committed:
<strong>#   (use "git rm --cached &#x3C;file>..." to unstage)
</strong>#         new file:   hello.py
</code></pre>

Now we're on the `main`branch, and we no longer have a warning about untracked files. Now Git is ready to commit, so let's make a new commit with a message:&#x20;

```bash
$ git commit --message "Added Hello World"
# [main (root-commit) e96ab34] Added Hello World
#  1 file changed, 1 insertion(+)
#  create mode 100644 hello.py
```

{% hint style="success" %}
You can use `git commit -m "your message"` and `git switch -c` as shorthand
{% endhint %}

The output here confirms your branch, commit ID (also called the **commit hash**), message, and what changed. Running `git status`confirms that our commit went through and that we're clear to start making more changes:&#x20;

<pre class="language-bash"><code class="lang-bash">$ git status
# On branch main
<strong># nothing to commit, working tree clean
</strong></code></pre>

Now that you've set up your repository and branch, the process will be the same for future changes: make changes, `git add`the files, then `git commit`with a message. Let's review that last commit

```bash
$ git show
# commit e96ab344b7b1a8499fcb01f9f39f943c31dc111e (HEAD -> main)
# Author: Levi Lesches <levilesches@gmail.com>
# Date:   Fri Apr 4 02:46:55 2025 -0400
# 
#     Added Hello World
# 
# diff --git a/hello.py b/hello.py
# new file mode 100644
# index 0000000..7df869a
# --- /dev/null
# +++ b/hello.py
# @@ -0,0 +1 @@
# +print("Hello, World!")
```

This has a lot of output, but you can clearly see the commit ID, author, date, message, which files were edited and even the diff for each file.&#x20;
