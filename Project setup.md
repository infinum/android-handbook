## Before starting work

1. Create a private git repository on Github/Bitbucket
2. Create an empty `master` and `dev` branch, mark them protected if possible
3. Set `dev` as the default branch
4. Ask the project manager for all the project documentation and design

## Initial project setup

1. Branch a new branch off `dev` (see [Git usage](/Git usage.md) chapter about details)
2. Create a new project in Android Studio using the wizard
3. Create gitignore file using [gitignore.io](https://www.gitignore.io/)
4. Set up buildTypes and flavors (usually you need a `staging` and `production` flavor)
5. Generate a development keystore with random passwords and add the configuration to `build.gradle`, the production keystore management is explained in the chapter [Keystore management](/Keystore management.md)
6. Add the static analysis and CI configuration as described in the `Continuous integration` chapter
7. Add an Application class and Timber, setup Timber in app class
8. Setup the basic networking and test code with dependency injection (see [Dependency injection](/Dependency injection.md) chapter for details)
9. Setup Fabric on Infinum or client account if available (make sure caught exceptions are also logged using [Timber](https://github.com/JakeWharton/timber))
10. Add app icons with a different icon for non-production server targets (we usually use a banner overlayed on the icon saying 'test' or 'staging')
11. Add the appropriate Proguard configuration depending on libraries used
12. Open a Pull Request to the appropriate colleague

The first pull request should contain only the initial setup code so it's easier to review.
