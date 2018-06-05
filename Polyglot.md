[Polyglot](https://polyglot.infinum.co/) is an in-house platform for managing translations.

The idea is to have all application strings in one place and share them between platforms. By using polyglot, translations across platforms are consistent and we remove the chance of developer error while copying the strings in the app.

You will need to login with your [Infinum Id](https://accounts.infinum.co/) and ask your team lead to grant you privileges on your projects.

## Usage for Android projects

To use Polyglot in and Android project, use the [Gradle plugin](https://github.com/infinum/Polyglot-Android-Client).

Consult the documentation on the link for more details.

Both the strings (files called `polyglot-v2.xml`) and the configuration file (`polyglot-config.yml`) should be committed to the repository so the app can be built even without access to the Polyglot API.

## Legacy polyglot

You may find older Android projects use a ruby gem called `polyglot-cli`. You will recognize such projects by the file `app/polyglot.yml` which is the configuration file.

All such projects should be migrated to the Polyglot gradle plugin linked above as the old API is deprecated and will be shut down in Q1 2017 (plans as of December 2016).

The migration is fairly easy:

1. Remove `app/polyglot.yml` and all `polyglot.xml` strings files
2. Add the Polyglot Gradle plugin and set it up as per docs, committing the config and translation files
