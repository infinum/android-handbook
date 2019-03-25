[Polyglot](https://polyglot.infinum.co/) is an in-house platform for managing translations.

The idea is to have all application strings in one place and share them between platforms. By using polyglot, translations across platforms are consistent, and we remove the chance of developer error when copying the strings in the app.

You will need to log in with your [Infinum ID](https://accounts.infinum.co/) and ask your team lead to grant you privileges on your projects.

## Using Polyglot in Android projects

To use Polyglot in an Android project, use the [Gradle plugin](https://github.com/infinum/Polyglot-Android-Client).

Consult the documentation on the linked page for more details.

Both the strings (files called `polyglot-v2.xml`) and the configuration file (`polyglot-config.yml`) should be committed to the repository so the app can be built even without access to the Polyglot API.

## Legacy polyglot

You may find that older Android projects use a Ruby gem called `polyglot-cli`. You will recognize such projects by the file `app/polyglot.yml`, which is the configuration file.

All such projects should be migrated to the above linked Polyglot Gradle plugin, as the old API is deprecated and will be shut down in Q1 2017 (plans as of December 2016).

The migration is fairly easy:

1. Remove `app/polyglot.yml` and all `polyglot.xml` string files
2. Add the Polyglot Gradle plugin and set it up according to the docs, committing the config and translation files
