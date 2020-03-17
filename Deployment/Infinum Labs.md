[Infinum Labs](https://labs.infinum.co/) is an in-house platform for deploying Android and iOS application builds.

The development process requires frequent app deploys (for project managers, testers and clients) so we needed a way to publish applications and enable access to them during the development and testing phase.

There are two ways you can upload your apk, via the Labs web app or by using the command line tool `shaman`.

Each Labs project has multiple environments defined, and each environment has a platform type (android/ios/...). Environments can be marked as **Developer only** which means they are not visible to clients - this is useful for internal testing.

Each build you to deploy to Labs needs to have a unique **versionName** within that project environment.

Each build should also contain a short change log (Markdown is supported).

## Shaman

You can install shaman by running `gem install shaman_cli`, provided you have a recent version of ruby installed.

For usage instructions consult [the docs](https://bitbucket.org/infinum_hr/gem-shaman).
