## Install basic tools

- [Android Studio](https://developer.android.com/studio/index.html)
- Git—`brew install git` ([brew docs](http://brew.sh/))
- Zeplin—`brew cask install zeplin` ([brew cask docs](https://caskroom.github.io/))

## Setup Java JDK

Java JDK comes embedded in Android studio and is often more compatible, so we don't need to manually download and install JDK. However, Android studio uses systems JDK by default, so the project structure needs to be changed in order to use the embedded one.

- On the welcome screen, go to Configure -> Default project structure... and change JDK location to Embedded JDK.
- If you have opened a project without following the first step, go to File -> Project structure... -> SDK Location and change JDK location to Embedded JDK. Then, go to File -> Other settings -> Default project structure... and change JDK location to Embedded JDK.

However, if you are going to use terminal for running some gradle tasks (and eventually you are going to) you may run in some problems if you are going to use Java version higher than 8. So I'd suggest downloading Java 8 and setting it to default in you terminal.

## Import code style

To make working with the rest of the team easier, we all use the same code style settings.
This ensures that there are no unnecessary changes caused by code style differences. It also makes it easier to conform to the established ckeckstyle rules. The code style includes Java, Kotlin, and XML styles.

To import Infinum's code style, follow these steps:

1. Download [code style file](https://github.com/infinum/android-handbook-private/blob/master/files/InfinumCodeStyle.xml)
2. Open Preferences -> Editor -> Code Style
3. Click on Manage next to Scheme picker
4. Import the downloaded file

Some options in our code style are **optional**:

- Java—Wrapping and Braces—Wrap on typing
- XML—Other—Wrap on typing

## Other useful macOS tools

- [iTerm2](http://iterm2.com/)
- [z](https://github.com/rupa/z/)—`brew install z`
- [Spectacle](https://www.spectacleapp.com/)—`brew cask install spectacle`
- [QuickLook plugins](https://github.com/sindresorhus/quick-look-plugins)
- [ImageOptim](https://imageoptim.com/mac)—`brew cask install imageoptim`
- flycut—`brew cask install flycut`
- Android file transfer—`brew cask install android-file-transfer`
- [alfi](https://github.com/cesarferreira/alfi)—`gem install alfi`
