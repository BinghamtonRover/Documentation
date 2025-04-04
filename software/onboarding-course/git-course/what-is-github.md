# What is GitHub?

Git is great when working on software by yourself. You get version control, feature branches, and a timeline full of commits that describe how your project evolved. But very quickly you'll find yourself working on software with others. We need a way to share repositories between developers, and that's exactly what **GitHub** is all about. GitHub allows you to control who can see your code, who can change your code, how pull requests are handled, and a lot more. In fact, pull requests are entirely a GitHub feature – Git just handles the merging. GitHub has many more community-centered features, but for this guide we will focus on how you can incorporate GitHub into your workflow.

## Local vs Remote

When you create a repository, it is said to be **local**. It only exists on your own device, in a specific folder. When you're ready to share or collaborate, you **push** your code to a hosted Git service like GitHub, and it becomes a **remote repository** that others can access by sharing the URL. The default name for this remote repository is `origin`. Similarly, when you create a branch, you are creating a **local branch**. When you're ready for a review or otherwise want to upload your changes, you push your branch to the remote repository, creating a **remote branch**.

For example, imagine creating a `Rover/` folder on your computer. You turn it into a local repository, then push it to GitHub and now it exists at `https://github.com/your-username/Rover.git`. There's a local `main` branch and a remote `main` branch, and they both have the same content. Then you create a feature branch, `branch-1`. This branch is local and is only on your computer until you push it, which creates a remote `branch-1`. Now you add a few more commits to your `branch-1`, making your branch **ahead** of the remote branch, so you push your changes up.

When changes occur on one end, the other end becomes out of sync. For example, while working on `branch-1`, someone else pushed a few commits to the remote `main` branch. Now your `main` branch is **behind** `remotes/origin/main`, and you must **pull** those commits and merge them with your own. Now, because you have made no changes to your own `main` branch, Git can do this automatically in a **fast-forward merge**. You're encouraged to try to pull `main` every time you sit down to code to stay up-to-date with the rest of your team.

{% hint style="warning" %}
By keeping your main branch clean of any of your own changes, you can completely avoid merge conflicts when pulling. But if you make a commit to `main`, even accidentally, you'll become _ahead_ of the main branch. If someone then merges a new feature, you've become both ahead _and_ behind. This will block you from pushing, and trying to pull will most likely result in a merge conflict. Being careful to always work in feature branches and never on the main branch directly is critical to avoiding situations like these.
{% endhint %}

When you're ready to submit your feature, you open a pull request. When your code is reviewed, it is merged into `main`. Now, your own local `main` branch is behind again, so you can perform another fast-forward merge to simply pick up those new changes and start the cycle again.

{% hint style="info" %}
It's simplest to think of local vs remote as "mine vs GitHub's". But there's nothing special about GitHub other than that it's accessible by other developers – so is the rover! When competing, we're forced to abandon GitHub and push and pull directly to the rover. As you can imagine, managing the local branches on each member's computer, the remote branches on GitHub, and the remote branches on the rover can be very complex and error-prone.
{% endhint %}

## More Resources

* [The Wikipedia page for GitHub](https://en.wikipedia.org/wiki/GitHub)
* [GitHub's Getting Started documentation](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git)
* [An intro to Git and GitHub for Python developers](https://realpython.com/python-git-github-intro)
* [our-github-organization.md](../../overview/our-github-organization.md "mention")

