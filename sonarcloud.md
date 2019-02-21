Sonarcloud is a continuous code quality tool which analyzes your implementation and notifies you about common pitfalls. The idea behind Sonarcloud integration is to keep the code base as clean as possible and to reduce the [technical debt](https://infinum.co/the-capsized-eight/is-your-business-sinking-into-technical-debt).

Sonarcloud support PR integration and it will block a PR from merging if the quality report is bellow our standards. Sonarqube and sonarcloud are the same thing. Sonarqube is a self hosted solution while sonarcloud is a cloud based code quality tool. There are small differences, but as far as we are concerned, they have the same features.

Before starting the integration, I suggest you ask your team lead to help you with it because you will need admin access for github and 1password.

## Initial integration

This integration does not take branches or PRs into the account. It doesn’t matter on which branch your are, on the sonarcloud you will see only 1 analysis. This means, if you run sonarcloud on a different branch, you will overwrite the previous analysis. However, you need to do this before setting up PR and branch analysis.

Here are the steps for initial integration:

1. Go to [sonarcloud.io](https://sonarcloud.io/) and login via github. Ask your team lead to give you **member access** to sonarcloud.

2. In the upper right corner click on the ”+” icon and select *Analyze new project*

3. Select *Setup manually* and set these parameters

	* **Organization** - Infinum infinum-android
	* **Project key** - Has to be unique so use your project package name
	* **Display name** - Use the same pattern as on github Android-{projectName}
	* **Private** - Make sure to select the Private option. Never select the public option!

4. Generate a token which will be used for analysis. Leave the generic name: *Analyze {projectName}*. 

5. Ask your team lead to save the token to 1Password. You can generate multiple tokens, so don’t worry if it gets lost, but in that case it you **have to tell it to your team lead and revoke it**. You can revoke tokens from your profile. 

6. You can now leave this page

7. In your project, checkout from your default to a feature branch.

8. Add a SONARCLOUD_TOKEN environment variable to your CI. On CircleCI go to **Project -> Settings -> Environment Variables -> Add variable**

9. In your `local.properties file` add a new variable `sonarcloud.token={generatedToken}`. Make sure `local.properties` is added to the .gitignore file.

10. Add sonarcloud plugin to the root `build.gradle` of your project

```groovy
buildscript {
    dependencies {
        classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:{sonarCloudLatestVersion}"
    }
}
```

11. In root of your project, add a folder named **config**. Inside of it, create a file called `sonarcloud.gradle`.

12. Copy paste this configuration to `sonarcloud.gradle` and fill in necessary data: 
* projectName 
* projectKey 
* lint report paths.

```groovy
apply plugin: 'org.sonarqube'

sonarqube {
    androidVariant 'stagingRelease’

    properties {
        property "sonar.projectName", {projectName}
        property "sonar.projectKey", {applicationId}
        property "sonar.androidLint.reportPaths", {…debug.xml}
        property "sonar.organization", "infinum-android"
        property "sonar.host.url", "https://sonarcloud.io"
        property "sonar.login", getSonarcloudToken()
    }
}

// Return sonarcloud token from the environment variable or from local.properties file
def getSonarcloudToken() {
    if (System.env.SONARCLOUD_TOKEN != null && System.env.SONARCLOUD_TOKEN != "") {
        return System.env.SONARCLOUD_TOKEN
    }

    Properties props = getLocalProperties()
    return props['sonarcloud.token']
}

def getLocalProperties() {
    Properties props = new Properties()
    if (file('../local.properties').exists()) {
        props.load(new FileInputStream(file('../local.properties')))
    }
    return props
}
```

13. In you application `build.gradle` add this line:
`apply from: '../config/sonarcloud.gradle’`

14. Build your project and run this command from terminal or Android studio:
`./gradlew sonarqube`

15. Check [sonarcloud](https://sonarcloud.io/organizations/infinum-android/projects) if project is analyzed. 

16. Add this command to circle CI config yml file. It has to be the last command, after static checkers and unit tests

```
  - run:
     name: Run SonarQube Analysis
     command: ./gradlew sonarqube -PdisablePreDex --console=plain
```

17. You are done. You can now commit and push the changes! Make sure the build passes on CircleCI.

## PR and branches integration

Sonarcloud has a strange way of working with branches. The branch which doesn’t have a `sonar.branch.name` property set is considered as the main branch and in the sonarcloud UI it will be shown as *Master*. The name can be changed in the project settings, but keep in mind that our master will be the *dev* branch since all our PRs are merged to it. The configuration which you will now setup sets the branch name or omits it for your default dev branch. The name of the default branch has to be hardcoded since default branch is a feature of github, not git.

Here are the steps to set PR and branches integration:

1. Copy paste these methods to your `sonarcloud.gradle` and change your default branch name. If you followed Infinum guidelines, it should be *dev* and you don’t need to change anything.

```
static def getDefaultBranch() {
    return "dev"
}

// Return CircleCI PR number extracted from CIRCLE_PULL_REQUEST environment variable
// CIRCLE_PR_NUMBER variable is available only for forked PRs
static def getCirclePRNumber() {

    if (System.env.CIRCLE_PULL_REQUEST != "" && System.env.CIRCLE_PULL_REQUEST != null) {
        return System.env.CIRCLE_PULL_REQUEST.tokenize("/").last()
    }

    return ""
}
// Returns current branch name or an empty string if current branch is the default one
// The default branch is shown as "master" on Sonarqube
static def getSonarqubeBranchName() {

    def currentBranch = getCurrentBranchName()

    if (currentBranch == getDefaultBranch()) {
        return ""
    }

    return currentBranch
}

// http://coders-kitchen.com/2013/11/01/gradle-git-how-to-map-your-branch-to-a-deployment-profile/
static def getCurrentBranchName() {
    def currentBranch = ""
    def proc = "git rev-parse --abbrev-ref HEAD".execute()
    proc.in.eachLine { line -> currentBranch = line }
    proc.err.eachLine { line -> println line }
    proc.waitFor()
    return currentBranch
}
```

2. Copy paste this code to your sonarqube properties in sonarcloud.gradle. Do not override the initial implementation, add this below default properties.

```
sonarqube {
    properties {
        if (getCirclePRNumber() != "") {
            // This is a PR so provider properties are set
            property "sonar.pullrequest.provider", "github"
            property "sonar.pullrequest.github.repository", "infinum/${System.env.CIRCLE_PROJECT_REPONAME}"
            property "sonar.pullrequest.github.endpoint", "https://api.github.com/"
            property "sonar.pullrequest.branch", "${System.env.CIRCLE_BRANCH}"
            property "sonar.pullrequest.key", getCirclePRNumber()
        } else {
            // If the branch is not the main branch on github it will set it's name, an empty string is set otherwise
            property "sonar.branch.name", getSonarqubeBranchName()
        }
    }
}
```

3. Ask your TL to go to **Github -> Infinum -> Settings -> Installed Github Apps -> Sonarcloud -> Configure** and add your project to sonarcloud app. Do not allow sonarcloud access to all projects, only the one you are adding.   

4. You can now commit and push. After the build passes on the circle CI, go to the project on sonarcloud and in the upper left corner check if you can switch between branches.

5. After you merge the sonarcloud integration, go to **Github -> Project -> Settings -> Branches -> Branch protection rules** and set the sonarcloud status check as required for your default branch. 
