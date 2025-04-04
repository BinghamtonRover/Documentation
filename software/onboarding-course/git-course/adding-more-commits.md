# Adding more commits

### Simple changes

Let's add a comment to the Python file:&#x20;

{% code title="hello.py" %}
```python
# Print a greeting to the user
print("Hello, World!")
```
{% endcode %}

To commit this, we need to simply add it to the stage and commit, like last time:

```bash
$ git add hello.py
$ git commit -m "Added a comment"
# [main c5ac38c] Added a comment
#  1 file changed, 1 insertion(+)
```

{% hint style="success" %}
This pattern of "change file, add file, commit file" is so common, Git has a shortcut: `git commit -a -m "your message"` will add all the files you've changed since the commit, so you don't need to run any `git add`commands. Note that this does _not_ add untracked (new) files, you'll still need to use `git add` for those.
{% endhint %}

### Adding new files

Let's make our Python script import a file:&#x20;

{% code title="data.py" %}
```python
GREETING = "Hello, World!"
```
{% endcode %}

{% code title="hello.py" %}
```python
import data

# Print a greeting to the user
print(data.GREETING)
```
{% endcode %}

```bash
$ git status
# On branch main
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   hello.py
# 
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         data.py
# 
# no changes added to commit (use "git add" and/or "git commit -a")
```

This time we got another section of output

> Changes not staged for commit

These files are being tracked by Git, but you haven't told Git you're ready to commit these files. Git wants you to explicitly **stage** your changes, meaning you have to tell Git which files you want to commit before committing them. Note that Git works on a file basis here, so you can't stage or commit only certain lines in a file at a time.&#x20;

```bash
$ git add hello.py
$ git status
# On branch main
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         modified:   hello.py
# 
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         data.py
```

And just like before, we'll add the new file so Git can track it:&#x20;

```bash
$ git add data.py
$ git status
# On branch main
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         new file:   data.py
#         modified:   hello.py
```

_Now_ we're ready to commit:&#x20;

```bash
$ git commit -m "Moved the greeting message to data.py"
# [main f53d7d2] Moved the greeting message to data.py
#  2 files changed, 4 insertions(+), 1 deletion(-)
#  create mode 100644 data.py
$ git status
# On branch main
# nothing to commit, working tree clean
```

{% hint style="success" %}
`git status -sb` gives a more concise output. We'll use that from now on, but feel free to run without it.
{% endhint %}

### Ignoring files

```bash
$ git status -sb
# ## main
$ python hello.py
# Hello, World!
$ git status -sb
# ## main
# ?? __pycache__/
```

What happened? We had a clean **working tree**, but when we ran the code, Python generated a cache that contained the compiled output of `data.py`. In general, compiling code will often produce files that you don't want to commit because they're not the actual source of truth; they can be derived from other files. We want to tell Git to ignore `__pycache__` and not track it. We can do this by creating a **gitignore file,** which is just a file called `.gitignore`

{% code title=".gitignore" %}
```gitignore
__pycache__/
```
{% endcode %}

```bash
$ git status -sb
# ## main
# ?? .gitignore
```

Now, Git is telling us we have a new file called `.gitignore` . but it no longer cares about `__pycache__` . It might feel weird to commit the file that tells Git what files to _not_ commit, but if you didn't, your fellow team members would run into the same problems when they run your code. Every project gets to specify which files should be ignored.&#x20;

```bash
$ git add .gitignore
$ git status -sb 
# ## main
# A  .gitignore
$ git commit -m "Added __pycache__/ to gitignore"
# [main 92eca49] Added __pycache__/ to gitignore
#  1 file changed, 1 insertion(+)
#  create mode 100644 .gitignore
```
