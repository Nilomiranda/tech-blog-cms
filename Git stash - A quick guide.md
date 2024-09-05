
When working on a development branch, there may be a time where you'll need to check out to another branch to do another piece of work or test something else. 

If you're not ready to commit the changes of your current work, you can use git stash to temporarily save these changes and go back to them once you're ready to resume the work you're doing.

Git stash itself has a lot of resources to work with. You can have a look at a more in-depth documentation [here](https://www.atlassian.com/git/tutorials/saving-changes/git-stash) for example.

But here are my essentials that I use constantly that allow me to get the most of the command. Call it the 20% of stash that allows you to do 80% of the work.

```bash
git stash
```

This will simply stash and save the work in progress in a temporary working directory.

When you're ready to go back to what you were working on, just:

```bash
git stash apply
```

to re-apply the changes from **the latest stash**

You can list the existing stashes, so you know what stash you want to apply. Useful in cases where you stashed multiple times and now you need to apply changes from a stash that not necessarily was the latest.

```bash
git stash list
```

Will show a list of stashes:

```bash
stash@{0}: stash message
stash@{1}: stash message
```

`stash message` will either be the auto message that it's created when you run `git stash`. 

To keep your stashes organized, so you know which one is about what, you can set your own message with:

```bash
git stash save "your custom message"
```

Then when you run `git stash list` again, you'll see something like:

```bash
stash@{0}: stash message
stash@{1}: your custom message
```

So to apply a specific, stash, say `stash@{1}` in this case you can:

```bash
git stash pop stash@{1}
```

to apply the stash **1** instead of the latest **0**

With these commands you can do a lot with the `stash` command and safely navigate between branches and temporary working without having to forcefully commit to not lose what you started working on.