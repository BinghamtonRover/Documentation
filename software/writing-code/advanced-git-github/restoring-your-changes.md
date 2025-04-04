# Restoring your changes

The simplest case is when you want to restore your _uncommitted_ changes:&#x20;

<pre class="language-bash"><code class="lang-bash">$ echo oops >> hello.py
$ git status -sb
<strong># ## main
</strong>#  M hello.py
$ git restore hello.py
$ git status -sb
# ## main
</code></pre>

If you've already staged the change, you'll need to restore twice:

```bash
$ echo oops >> hello.py
$ git add hello.py
$ git status -sb
# ## main
# M  hello.py
$ git restore --staged hello.py
# ## main
#  M hello.py
$ git restore hello.py
$ git status -sb
# ## main
```

{% hint style="info" %}
Note the position of the `M`under the two `#`s, as these indicate whether the file is staged. You should also see a red `M`for unstaged and a green `M`for staged. As always, if you're confused, run `git status`without `-sb`for more information.
{% endhint %}

If you've already committed a change, you can revert it:&#x20;

```bash
$ git log 
# Find the hash of the commit you want to revert
$ git revert COMMIT_HASH
```

{% hint style="success" %}
For any of these commands, you can replace the commit hash with an expression like `HEAD~1`, which means one commit before the current state (the **head** of the tree). You can change this number to go further.
{% endhint %}

If you want to restore a file to how it was at a certain point in time, that's also possible:

<pre class="language-bash"><code class="lang-bash">$ git log 
# Find the hash of the commit you want to restore your file to
$ git restore --source COMMIT_HASH hello.py
$ git status -sb
<strong># ## main
</strong>#  M hello.py
$ git commit -a -m "Reverted hello.py back to earlier commit"
# [main b99e42c] Reverted hello.py back to earlier commit
#  1 file changed, 1 deletion(-)
</code></pre>

Note that this just changes the file. You're still on the hook to stage and commit the change, as usual. If  you want to revert the entire repository to how it was at a certain commit, you can do that too:&#x20;

```bash
$ git log 
# Find the hash of the commit you want to go back to
$ git reset COMMIT_HASH
$ git commit -a -m "Reset back to a known working state"
```

So far, all of these commands keep the history intact, and simply add new commits to undo older ones. By adding `--hard`, you can actually take the repository back to an earlier commit, _discarding all commits since then._

```bash
$ git reset --hard c5ac38c 
# HEAD is now at c5ac38c Added a comment
```

It should go without saying a command like this is very dangerous, and you can very quickly lose a lot of work if not done correctly. There are times when this is necessary, however, like when you've accidentally committed to the `main`branch and need to restore it back to the last commit on GitHub. In cases like this, you can first create a temporary branch and then perform this command on `main` , leaving your commits intact on the new branch.&#x20;

{% hint style="danger" %}
Pushing is forever. If you've pushed a commit, no amount of resetting locally can undo that, and you can't change history on remote repositories. Never reset a commit with `--hard`that you've already pushed to a remote branch. &#x20;
{% endhint %}

