# Merge Conflicts

One side effect of trunk-based development, and really Git in general, is that over time we end up with many small branches for unrelated features. Usually, this is a good thing: if `featureA` touches file `a`.py, and `featureB` creates `b.py`, then both can be merged into `main` by applying their commits in any order. That's a very ideal case, and in fact we try to structure our code to allow this to happen as often as possible. But what if things get more complex?

Sometimes, when merging code between multiple branches, you'll get what's called a merge conflict. Specific advice on dealing with them will be in the following pages, but the reason why they happen is actually pretty simple. If you have a function `buggy()` that is modified by two branches, then the first will merge into `main` cleanly while the next will cause a **merge conflict**. These are annoying, but fundamentally simple to resolve: you or your team lead will go through each of the conflicting files, review the diffs from both branches and use context to decide which lines should be accepted. Usually, the end result will be some combination of both diffs.

Merge conflicts can be pretty nasty. This page will help you understand what they are and how to fix them, while [avoiding-merge-conflicts.md](../../writing-code/advanced-git-github/avoiding-merge-conflicts.md "mention") can tell you how to avoid them in general.

## Resolving conflicts

Consider the following function:

```python
def greet(): print("helo world")
```

Someone on the team noticed that the word `hello` was misspelled. So they fork and add a commit:

```diff
- def greet(): print("helo world")
+ def greet(): print("hello world")
```

{% hint style="info" %}
This is called **diff syntax**, and it's how diffs are represented. Git works in lines, not words, and only tracks additions and deletions, not _changes_. So, to change one word, Git deletes the entire line and adds an almost identical copy, with one word changed.
{% endhint %}

Meanwhile, someone else noticed that the punctuation was all wrong, but they missed the typo:

```diff
- def greet(): print("helo world")
+ def greet(): print("Helo, World!")
```

Trying to merge both of these diffs will cause a conflict because they touch the exact same line. Git can't just keep one diff and throw away the other, because both offer valuable contributions to the code in their own way. To resolve the conflict, one would review both diffs, understand what each one was trying to do, and make a new commit that represents both of those changes:

```diff
- def greet(): print("helo world")
+ def greet(): print("Hello, World!")
```

Merge conflicts can happen when merging feature branches into `main`, but they can also happen the other way around. When working on your feature branch, you should regularly update your `main` branch to check for changes made by other members, and merge those changes into your feature branch. If someone else touched the same file as you did, that's a conflict. Understanding what to do is vital for resolving them while not losing valuable work.

In reality, conflicts will usually not be this trivial. Imagine someone refactoring a block of code while another renames a function used in those lines. Or someone moving a file that someone else was working on. Or two features that both add critical logic to the same function. While Git is smart and can sometimes handle these, it's usually up to the authors and reviewers to come to analyze the specific case and decide what the merged result will look like.

## More Resources

* [The GitHub documentation on merge conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/about-merge-conflicts)
* [Atlassian's article on merge conflicts](https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts)
* [This Stack Overflow question about merge conflicts](https://stackoverflow.com/questions/24852116/what-exactly-is-a-merge-conflict)
