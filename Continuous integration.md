Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

By integrating regularly, you can detect errors quickly and locate them more easily.

## Bitrise

For all active projects we use Bitrise (previously we used CircleCI). Unlike Circle, which is suitable for any type of project, Bitrise is CI/CD just for mobile apps and it offers [some advantages](https://infinum.com/the-capsized-eight/bitrise-vs-circleci-for-android-in-a-head-to-head-battle) for mobile development CI.

Bitrise and Circle are a cloud-based solution, your code gets cloned to a remote server, which could pose a security problem for your client.

## Configuration

We usually have two main workflows:

- One that's run for every push and contains all the checks and linters. Status of this workflow is reported back to Github, so Pull Request can't be merged if these checks don't pass. That workflow might look something like this:

![aa](https://i.imgur.com/mv0fCWS.png)

- One that's run to distribute the app to internal store or Google Play


## Protected branches

In the git flow model that is applied in Infinum, the `dev` and `release` branches have a special role in the development and release cycle. It is expected that the code is not pushed directly to those two branches; therefore the branches need to be set as protected.

First, the `dev` and `release` branch need to be set as protected. In order to set them as protected, administrator rights are required on the working project. The administrator user can click on the Settings button, as depicted in the picture

![Click on the Settings](/img/CI-protect-branch-click-setting.png)

Next, select the Branches tab as shown in the picture.

![Click on the Branches](/img/CI-protect-branch-click-branches.png)

Next, select the `dev` branch in the `Protected branches` section.

![Select the branch](/img/CI-protect-branch-select.png)

Mark the check boxes as shown.

![Mark check boxes](/img/CI-protect-branch-check.png)
