Before you can start using Git, you need to [generate an SSH key](https://help.github.com/articles/generating-ssh-keys/) and add the public part of the key to your GitHub and Bitbucket accounts.
In that way, you can interact with private repositories without entering your username and password all the time.
Just remember to clone the repository using the [SSH clone URL](https://help.github.com/articles/which-remote-url-should-i-use/#cloning-with-ssh).

## Git Flow

We use a variant of the [Git Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) workflow. The difference between our variant and the linked document is that we don't use release branches.
You can find a more detailed explanation of Git Flow [here](http://nvie.com/posts/a-successful-git-branching-model/).

![Git Flow](/img/git-flow.svg)

There are two main branches, `master` and `dev`.
The `master` branch contains **only published code** at all times, and each release is **tagged** with the version number.
The `dev` branch serves for integration of various features/fixes developed in separate branches.

## Git tagging

Before releasing the app on PlayStore, the release has to be tagged with the version number. Also, provide a short description for the release. Here's an example of adding a tag:

```bash
git tag -a 3.0.0 -m “The best version ever“
git push --tags
```
You can find more info about tagging [here](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

### Naming conventions

The development branch should always be named `dev`, for consistency across projects.  

Branches are named according to their use, and words are delimited by dashes. Some examples:

    feature/login-screen
    enhancement/ui-improvements
    fix/533-pin-user-alert

### Starting work on a new feature

```bash
git checkout dev
git checkout -b feature/login-screen
```

### Merging the feature into `dev`

The preferred way of merging a branch back into `dev` is by creating a pull request and assigning it to a colleague for review.

There are two different roles in every pull request—the assignee and the reviewer. Reviewers are your colleagues who review your pull request, whereas the assignee is the person responsible for merging or cancelling the pull request, in other words, **you**.

If the branch cannot be merged back into `dev` because of a merge conflict, you need to fix the conflict on your feature branch by either rebasing from `dev`

```bash
git checkout feature/login-screen
git rebase dev
```

or by merging `dev` into your branch

```bash
git checkout feature/login-screen
git merge dev
```

### Commit early, commit often

Don't forget to commit and push your code to the remote repository, this will protect you against losing your hard work.
Make small commits with meaningful commit messages and push at least once a day.

### Don't commit generated code

Generated code does not belong in the repo. Use the .gitginore file to exclude it. This file can be generated using the [gitignore.io](https://www.gitignore.io/) tool. All you need to do is write a few keywords (such as MacOS, Android, IntellJ, Android Studio), and the tool will generate the file content for you.

### Don't commit chunks of commented out code

Don't comment out code and commit it to the repository so you can 'uncomment it if you ever need it again'. This creates a mess in the code and makes it less readable. It's easy to return to an earlier commit and look up the code that you removed, so there is no need to leave commented code lying around everywhere.

### Resolving merge conflicts

![Resolving conflicts in Android Studio](/img/idea_vcs_magic_resolve.png)

Android Studio has a great built-in tool for resolving merge conflicts.
You can find it in `VCS` — `VCS Operations Popup`.
Before you can use this tool, select Git as your VCS for your project under `Preferences` — `Version Control`.
You can find more details about the tool [here](https://www.jetbrains.com/idea/help/resolving-conflicts.html).

### Quick trick

For a quick overview of your commit messages for today and yesterday, you can use the following aliases:

```bash
alias today='git log --since=6am --format="* %s" --author="$(git config user.email)" --reverse -- | pbcopy'
alias yesterday='git log --since=yesterday.6am --until=6am --format="* %s" --author="$(git config user.email)" --reverse -- | pbcopy'
```
