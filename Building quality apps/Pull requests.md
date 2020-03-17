Pull requests are the essential part of our workflow. This section describes how to create and review them.

### General rules
- A pull request cannot be merged until the CI build passes.
- The `dev` and `release` branches should be set as protected, which means no one can force push into them, and they can't be deleted. If you are not sure how to set them as protected, or you don't have access, ask one of the team leads to do it for you.

### Creating a pull request

1. A good pull request includes only one feature or a bugfix. Fixing multiple issues or adding several features in a pull request is not good practice, and it should be avoided whenever possible.
2. When creating a pull request, make sure you write a good description. It doesn't have to be long, but it has to provide context to the reviewer. Commit messages will not be considered a description of what has been done in the pull request.
3. If possible, provide the URL of the task which describes the feature or the issue.
4. Review your own pull request before creating it. By reviewing your own pull request, you get a chance to notice some obvious stuff before assigning it to a reviewer.

### Reviewing a pull request

1. When reviewing a pull request, make sure to check how the changes fit the current code base and architecture. Do not approve the pull request if the changes are not aligned with the rest of the codebase for no obvious reason.
2. In addition to code inspection, check out the branch on your machine and test it on your mobile phone. Use common sense for this. It is not important to test the changes if a pull request changes only one file, but in case of some bigger features or fixes, you should test the implementation. The idea behind this is to catch some obvious issues (paddings, margins, user-specific bugs, etc.) before they reach our testers.

These are guidelines, not rules, so use common sense when creating or reviewing a pull request.
