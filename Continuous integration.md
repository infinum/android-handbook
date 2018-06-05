Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

By integrating regularly, you can detect errors quickly, and locate them more easily.

## Circle CI

Circle CI is a container-based CI service. It has multiple features:

* support for all kinds of programming languages
* free of charge for first container
* support for static code checkers (Lint, Findbugs, PMD, Checkstyle)
* no additional configuration is needed
* everything is maintained and updated by their support team

On the other hand, Circle CI currently only supports Github repositories and it uses default Android emulator, so your execution times are a little bit slower than expected.  You'll also need admin privileges to Github's repository which you want to connect to Circle CI.

Also, as Circle CI is a cloud based solution, your code gets cloned to remote server which could pose a security problem for your client.

## Configuration

Circle CI uses simple configuration script, which is placed in `.circleci/config.yml` file:

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
        - master
        - labs
        - dev
```

You also need to add the following code in yout top-level `build.gradle` (the one in the root of the project) to [disable pre-dexing](http://tools.android.com/tech-docs/new-build-system/tips#TOC-Improving-Build-Server-performance):

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

This will significantly speed up compilation of your apps on CircleCI.

## Protected branches

In the git flow model that is applied in Infinum, `dev` and `master` branches have a special role in the development and release cycle. It is expected that the code is not pushed directly to those two branches, therefore the branches need to be set as protected. Also, in order to make sure that the code in those two branches builds successfully at any time and always satisfies statical analysis rules and any other rules defined in `circle.yml`, Circle CI needs to be enabled on `dev` and `master`.

First, `dev` and `master` branch need to be set as protected. In order to set them as protected, an administrator rights are required on the working project. Administrator user can click the Settings button, as depicted in the picture

![Click on the Settings](/img/CI-protect-branch-click-setting.png)

Next, select the Branches tab as shown in the picture.

![Click on the Branches](/img/CI-protect-branch-click-branches.png)

Next, select the `dev` branch in the `Protected branches` section.

![Select the branch](/img/CI-protect-branch-select.png)

Mark the check boxes as shown.

![Mark check boxes](/img/CI-protect-branch-check.png)

Do the same for the `master` branch.
