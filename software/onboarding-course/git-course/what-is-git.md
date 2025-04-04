# What is Git?

## Version control

When making changes to an important project, called a **repository**, you might be tempted to copy your folder and start making your changes to the `v2`, leaving the original files intact in case you need to revert back. And then a `v3`, a `v3_final`, and so on. These changes quickly become cumbersome to navigate, and questions like "what changed between versions" become impossible to answer. Not to mention, all these versions take up _space_ on your device.

Git is a version control tool that offers solutions to all this and so much more. Instead of keeping track of files, Git tracks _changes_, which we call **diffs**. By keeping just the differences between versions, Git can very efficiently store many versions for many files. Each diff can be applied and unapplied as needed to essentially travel back and forth through a given file's history. A **commit** contains the diffs to all the files you've changes between versions, as well as a descriptive message about your change. So when you want to go back to an earlier version, you go back to an earlier _commit_, and Git plays the diffs in reverse.

As you keep making changes, adding functions and fixing bugs, your commits build a **history**. If a bug is introduced, you can easily jump through your timeline to find the exact commit that introduced the bug (by seeing which commit goes from clean to broken), and you can look through the diffs to find the exact lines of code that caused the bug.

## Branches

Versions aren't just for tracking potentially dangerous changes. Say you want to work on a complex feature while also fixing bugs that come up in production. You'd like a way to start writing commits for that feature that don't affect the "main" timeline – at least, not until you've finished. Git offers **branches**, completely separate timelines for your code that can have very different commits.

When working on your project, Git creates an implicit **main branch** that all your commits go to. Later, when you decide to **branch off**, Git remembers which commit you are branching off at. Now you can continue adding commits to the main branch, but your new branch will not see those. Instead, the new branch starts at the commit you branched from, and all new commits on the new branch do not affect the main branch. Think of it as a fork in the road – you can continue on `main`, or on your new branch.

There are different conventions, but we use the **trunk-based development method**. This means that we treat `main` as the canonical copy of the code – at any given point, it's most likely that the rover is using a commit from the main branch. We therefore have very strict requirements for the main branch, such as automated testing (CI) and manual code reviews from your team lead. When making changes, you create a new **feature branch** and work on that, pulling in updates from `main` when there are any. When you're done, you open a **Pull Request**, which signals to your team lead that you want them to pull in your feature branch.

Using this method means that the main branch is always in a tested and finished state. It might not have the newest and flashy features, but anything on `main` has been reviewed, passed unit tests, and usually tested on real hardware. This allows us to work on multiple complex tasks while still having complete confidence in the code actually running on the rover.

## More resources

* [The Wikipedia page on Git](https://en.wikipedia.org/wiki/Git)
* [The Git website's explanation](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git)
* [Atlassian's explanation of Git](https://www.atlassian.com/git/tutorials/what-is-version-control)
* [This MIT course on Git](https://missing.csail.mit.edu/2020/version-control/)
* [The Trunk-Based Development Site](https://trunkbaseddevelopment.com/)
* [Atlassian's explanation of Trunk-Based Development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development)
