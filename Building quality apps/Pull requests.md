There are two types of [collaborative development models](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-collaborative-development-models) when using Git. We use the *shared repository model*, which means collaborators are granted push access to a single shared repository. Software is developed in branches that branch off `master`, which are pushed to the repository. We merge these branches into `master` by opening what's called a *pull request*, or PR for short.

You should keep branches small. The smaller they are, the easier they are to grasp, and therefore both to describe and review in a pull request. This leads to better **code quality assurance**, which is the main goal of pull requests.

If you're a junior developer, pull requests are also a great way to learn from your senior colleagues and improve your development knowledge and skills. If you're not a junior developer, but are new to the company, they're a great way to get acquainted with the company-specific conventions. If you're an intermediate or a senior developer, through pull requests, you allow your colleagues to offer a fresh set of eyes and have a chance to hear their valuable opinion on the matter at hand.

## Writing a PR description

You want to make it as easy as possible for your colleagues to grasp the changes and offer their view. You can do this by providing a useful description. As an author of the PR, you have a good mental model of everything that's happening in it. Your goal is to help your reviewer build that mental model.

Links to tasks can be helpful, but they are very far from enough. Even if the task is clearly defined and easy to understand, which is not always the case, tasks are more useful to the developer than the reviewer. A task specifies what needs to be done, but the reviewer needs to review the *implementation* of it, and the description of the *implementation* needs to exist.

A task may require certain contextual knowledge in order to be properly understood, as well. One other important goal of the description is to provide the background and define or explain the project-specific, task-specific, or PR-specific terminology.

Links or screenshots of the UI design, and potentially screenshots of its implementation can be very helpful.

Don't use artificial structures similar to grouping code by layer or type. An example of this would be a 'Features' section and a 'Fixes' section. We don't do that because it's much more useful to group related things together. Such structures also encourage using short bulleted lists of short single-sentence items that provide little to no value. Write full sentences and paragraphs. Writing is a valuable skill to be improved on. Use your own reasoning on a case-by-case basis, of course. Sometimes it's okay to have a bulleted list.

### Contextualise and group changes together

When the reviewer opens the 'Files changed' tab in the pull request, they get an **unordered** list of files changed, with **unordered** changes inside them. They'll have a hard time figuring out what's going on by looking at what's basically a random list of changes.

There're probably several groups of related changes working towards different goals. Connected goals, maybe, since they are in the same PR, but different. Sometimes they may not even be connected, which is an indicator that there probably should have been another PR, but it happens.

Changes being in the same file doesn't necessarily mean they are related to the same goal.

Looking at the individual commits can somewhat help the situation. However, this is not the solution. Even if your commits are small, focused, and have clear messages, they still lack the big picture which needs to be deduced by the reviewer.

Commits are not always in a helpful order, as well. For example, one commit may introduce a change aimed at one goal, then the next commit might take a small detour to introduce a change related to another goal, or some other side-thing.

Further, say there are 40 commits. (This shouldn't be considered a large number of commits, by the way.) At commit 25 you figured out that the thing you did at commit 14 needs to be undone, and at commit 29 you changed the way something at commit 17 is implemented.

The proper way of dealing with this is grouping the related changes together in the description. Because you already have a mental model, going through the unordered list of changes is not a problem for you, as you can easily recognise them.

### The algorithm

Duplicate the browser tab which contains the PR. One will be for writing the description. Another will be for going through the 'Files changed'. If you have multiple displays, even better. Put the tab in which you'll write a description on one display, and the tab with the changes on the other. This will make it easier for you to switch between writing a description and looking at the changes.

Go through the files changed one-by-one, and go through the change hunks one-by-one.

That change you're looking at belongs to a group of changes aimed towards a goal. Does your description contain a section for this goal? If not, create a section. Use [markdown](https://guides.github.com/features/mastering-markdown/) to your advantage.

Describe the change inside this section. Perhaps it's already described, perhaps you need a single sentence, perhaps you need a paragraph, perhaps you need to reword what you wrote previously. If there is a relevant docstring, it can simply be pointed to in the description.

Not sure what to do with the change you're looking at right now? Skip it and come back to it later.

Finished with all the changes in a file? Collapse the file to get it out of your way and make it easier to navigate and see what's left.

After you're done, give it a last couple of re-reads and re-iterations.

### Docstrings and GitHub Wiki

Now that you've written a PR description, think about whether it would be useful to have some of this information in a more accessible medium, such as docstrings or a GitHub Wiki page of the project. This information can then be pointed to in the PR description instead.

## Finally

Every PR requires an individual approach, and always apply your own reasoning on a case-by-case basis. Giving an algorithm a go and seeing what comes out should be no problem, though. 

After reading your description, the reviewer should be able to have a single pass through the 'Files changed' section, without going back and forth hopelessly trying to make sense of the changes and hopelessly trying to build a mental model.

The title should follow the same format as a commit message heading. That is, use the imperative mood, as if giving a command, and keep it short. You can leave the branch name, if you choose to, but it takes 10 seconds to think of something better. The title is far less important than the description, though.
