## Before starting work

1. Ask your Tech/Team leader to create a private Git repository on Github/Bitbucket.
2. If you are going to use `release` branches, add protection regex with same rules as for the `master` branch.
3. Ask the project manager to provide you with all project documentation and design.

## Initial project setup

1. Branch a new branch off `master` (see the [Using Git](/books/android/building-quality-apps/using-git) chapter for details).
2. Create a new project in Android Studio using the wizard.
3. Create a gitignore file using [gitignore.io](https://www.gitignore.io/).
4. Set up buildTypes and flavors (you usually need a `staging` and `production` flavor).
5. Generate a development keystore with random passwords and add the configuration to `build.gradle`. The production keystore management is explained in the [Keystore management](/books/android/common-android/keystore-management) chapter.
6. Add static analysis and CI configuration as described in the `Continuous integration` chapter.
7. Add an Application class and Timber, set up Timber in the app class.
8. Set up basic networking and test code with dependency injection (see the [Dependency injection](/books/android/project-structure/dependency-injection) chapter for details).
9. Set up Firebase on the Infinum or client account if available (make sure that caught exceptions are also logged using [Timber](https://github.com/JakeWharton/timber)).
10. Add app icons with a different icon for non-production server targets (we usually use a banner overlay on the icon saying 'test' or 'staging').
11. Add the appropriate ProGuard configuration depending on the libraries used.
12. Create README.md (see the [Readme](/books/android/internal-section/project-readme) chapter for details). 
13. Open a pull request to the appropriate colleague.

The first pull request should contain only the initial setup code so that it's easier to review.
