Being mindful over the code quality is a desirable trait in every developer. At Infinum, code quality matters. With that in mind, we have devised a few general rules which every Android developer should follow.

### Lint

Lint is a static code checker and is used to indicate suspicious and potentially harmful lines of code that do **not** follow certain style guidelines.

Lint should always be enabled on release builds, in other words, every build that is given into the hands of our internal QA team or the client **must** pass lint and other static code checkers on CI.

A situation could arise where you are working on an older project with outdated codebase, in which case, it's expected of you to set up static code checkers, if they aren't already, and create a [baseline](https://sites.google.com/a/android.com/tools/tech-docs/lint-in-studio-2-3#TOC-Creating-a-Baseline) if needed. The entire new codebase that's written in the project **should pass** static code checkers.

### Proguard

Every build that's sent to the QA team or clients **must** be obfuscated with Proguard. In other words, only release builds should be sent which have Proguard enabled in `build.gradle` file via `minifyEnabled` flag set to `true`.

The only exception to this rule should be a situation where the QA team or the client specifically asks for an unobfuscated build, in which case you should comply.

### Crashlytics

Every build that's uploaded to Infinum Labs **should**:

* **not** be debuggable
* pass static code checkers
* be obfuscated with Proguard
* have Crashlytics enabled

If a member of the QA team wants to see API responses and requests you are supposed to setup [Stetho](https://github.com/facebook/stetho) or [Chuck](https://github.com/jgilfelt/chuck) for **staging** builds.

### Prince Of Versions

We have developed a library that checks for updates of an application using a configuration file and it's called - [Prince of Versions](https://github.com/infinum/Android-Prince-of-Versions).

Using this library is a requirement on every project, old or new. If you ever stumble upon a project that doesn't have Prince of Versions set up you should contact your Team Lead or Project Manager and let them know of the issue. They should be able to provide you with a few hours to set up Prince of Versions.

### Test what you develop

If you implement a feature, it's **required** of you to test it out before you send it to the QA team or clients. Once the build is deployed it's expected that the app doesn't crash on startup or has similar issues which would indicate the laziness of the developer.

We're **not expecting deploys of bug-less builds**, but using common sense and testing out what you implemented should be in your every day routine.

### Architecture planning

If ever in doubt about the architecture you're about to implement it's always a good idea to ask someone else's opinion on the matter. Having more than one opinion broadens your views and should teach you a couple of things:

* how to avoid hidden pitfalls which you haven't thought of yourself
* alternative ways to fix a specific problem
* you could be biased and not aware the fact that your solution to the problem might not be the best one out there
* two minds are more powerful than one

### Communication is key

There could be situations where you are not completely certain what to do next or you find yourself in a bit of a loop. If such an event does occur, keep in mind that you have many colleagues who have been in the same situation you're in. Asking their advice will save you some time, and will in all probability result in gathering more information and clarity on the issue at hand.
