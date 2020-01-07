Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

By integrating regularly, you can detect errors quickly and locate them more easily.

## CircleCI

CircleCI is a container-based CI service. It has multiple features:

* support for all kinds of programming languages
* free of charge for the first container
* support for static code checkers (Lint, FindBugs, PMD, Checkstyle)
* no additional configuration needed
* everything is maintained and updated by their support team

However, CircleCI currently supports only GitHub repositories, and it uses the default Android emulator, so your execution times are a little bit longer than expected.  You'll also need admin privileges to the GitHub's repository to which you want to connect CircleCI.

Also, since CircleCI is a cloud-based solution, your code is cloned to a remote server, which could pose a security problem for your client.

## Configuration

CircleCI uses a simple configuration script, which is placed in the `.circleci/config.yml` file:

```yml
# Build configuration file for Circle CI
version: 2
jobs:
  build:
    working_directory: ~/code
    docker:
      - image: circleci/android:api-26-alpha
    steps:
      - checkout
      - restore_cache:
          key: jars-{{ checksum "build.gradle" }}-{{ checksum "app/build.gradle" }}
      - run:
          name: Install Build Tools
          command: echo y | sdkmanager "build-tools;26.0.2"
      - run:
          name: Download Dependencies
          command: ./gradlew androidDependencies
      - save_cache:
          paths:
            - ~/.gradle
          key: jars-{{ checksum "build.gradle" }}-{{ checksum "app/build.gradle" }}
      - run:
          name: Run Android Lint
          command: ./gradlew lintStagingDebug -PpreDexEnable=false -PdisablePreDex --console=plain
      - run:
          name: Run PMD
          command: ./gradlew pmd -PpreDexEnable=false -PdisablePreDex --console=plain
      - run:
          name: Run Checkstyle
          command: ./gradlew checkstyle -PpreDexEnable=false -PdisablePreDex --console=plain
      - run:
          name: Run Findbugs
          command: ./gradlew findbugs -PpreDexEnable=false -PdisablePreDex --console=plain
      - run:
          name: Run Tests
          command: ./gradlew testStagingDebug -PpreDexEnable=false -PdisablePreDex --console=plain
      - store_artifacts:
          path: app/build/reports
          destination: reports
      - store_artifacts:
          path: app/build/outputs
          destination: outputs
      - store_test_results:
          path: app/build/test-results
      - deploy:
          name: Labs
          command: |
            if [ "${CIRCLE_TAG}" != "" ]; then
              ./gradlew clean assembleStagingDebug -PpreDexEnable=false -PdisablePreDex --console=plain
              sudo apt-get install build-essential
              sudo apt-get install rubygems
              sudo apt-get install ruby-dev
              sudo gem install bundler
              ./config/deploy.sh
            fi
deployment:
  labs:
    tag: /labs\/.*/

experimental:
  # this will enable chat notifications only for certain branches
  notify:
    branches:
      only:
        - labs
        - dev
```

You also need to add the following code in your top-level `build.gradle` (the one in the root of the project) to [disable pre-dexing](http://tools.android.com/tech-docs/new-build-system/tips#TOC-Improving-Build-Server-performance):

```gradle
project.ext.preDexLibs = !project.hasProperty('disablePreDex')

subprojects {
    project.plugins.whenPluginAdded { plugin ->
        if ("com.android.build.gradle.AppPlugin".equals(plugin.class.name)) {
            project.android.dexOptions.preDexLibraries = rootProject.ext.preDexLibs
        } else if ("com.android.build.gradle.LibraryPlugin".equals(plugin.class.name)) {
            project.android.dexOptions.preDexLibraries = rootProject.ext.preDexLibs
        }
    }
}
```

This will significantly speed up the compilation of your apps on CircleCI.

## Protected branches

In the git flow model applied at Infinum, the `dev` and `release` branches have a special role in the development and release cycle. It is expected that the code will not be pushed directly to those two branches. Therefore, the branches need to be set as protected. Also, in order to make sure that the code in those two branches builds successfully at any time and always satisfies the static analysis rules and any other rules defined in `circle.yml`, CircleCI needs to be enabled on `dev` and `release`.

First, the `dev` and `release` branch need to be set as protected. In order to do so, administrator rights are required on the working project. The administrator user can click on the Settings button, as depicted in the picture.

![Click on Settings](/img/CI-protect-branch-click-setting.png)

Next, select the Branches tab as shown in the picture.

![Click on Branches](/img/CI-protect-branch-click-branches.png)

Next, select the `dev` branch in the `Protected branches` section.

![Select the branch](/img/CI-protect-branch-select.png)

Mark the check boxes as shown.

![Mark the check boxes](/img/CI-protect-branch-check.png)
