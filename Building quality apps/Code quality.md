Being mindful of code quality is a desirable trait of every developer. At Infinum, code quality matters. With that in mind, we have come up with a few general rules that every Android developer should follow.

### Lint

Lint is a static code checker used to indicate suspicious and potentially harmful lines of code that do **not** follow certain style guidelines.

Lint should always be enabled on release builds. In other words, every build that is given into the hands of our internal QA team or the client **must** be run through lint and other static code checkers on CI.

For example, you might be working on an older project with an outdated codebase. In that case, if static code checkers haven't already been set up, it's expected of you to set them up and create a [baseline](https://sites.google.com/a/android.com/tools/tech-docs/lint-in-studio-2-3#TOC-Creating-a-Baseline) if necessary. The entire new codebase that's written in the project **should pass** static code checkers.

### Proguard

Every build that's sent to the QA team or clients **must** be obfuscated with Proguard. In other words, only the release builds which have Proguard enabled in the `build.gradle` file via `minifyEnabled` flag set to `true` should be sent.

The only time you should make an exception to this rule is when the QA team or the client asks specifically for an unobfuscated build, in which case you should comply.

### Crashlytics

Every build that's uploaded to Infinum Tryout apps **should**:

* **not** be debuggable
* pass static code checkers
* be obfuscated with Proguard
* have Crashlytics enabled

If a member of the QA team wants to see API responses and requests, you are supposed to set up [Stetho](https://github.com/facebook/stetho) or [Chuck](https://github.com/jgilfelt/chuck) for **staging** builds.

### Prince of Versions

We have developed a library that checks for application updates using a configuration file. It is called the [Prince of Versions](https://github.com/infinum/Android-Prince-of-Versions).

Using this library is required on every project, old or new. If you ever stumble upon a project that doesn't have the Prince of Versions set up, you should contact your team lead or project manager and let them know. They should be able to provide you with a few hours to set up the Prince of Versions.

### Test what you develop

If you implement a feature, you are **required** to test it before you send it to the QA team or clients. Once the build has been deployed, it's expected that the app doesn't crash on startup or have similar issues which would suggest that the developer was lazy.

We **don't expect deploys of bugless builds**, but using common sense and testing what you've implemented should be a part of your everyday routine.

### Architecture planning

If ever in doubt about the architecture you're about to implement, it's always a good idea to ask for someone else's opinion on the matter. Having more than one opinion broadens your views and should teach you a couple of things:

* how to avoid hidden pitfalls which you haven't thought of yourself
* alternative ways to fix a specific problem
* you could be biased and not aware of the fact that your solution to the problem might not be the best one out there
* two minds are more powerful than one

### Communication is key

There could be situations where you are not completely certain what to do next, or you find yourself in a bit of a loop. If this happens, keep in mind that you have many colleagues who have been in the same situation you're in. Asking for their advice will save you some time, and will probably result in gathering more information and having a clearer perspective on the issue at hand.
