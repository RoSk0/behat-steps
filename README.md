# Behat Steps
Collection of Behat steps for Drupal

[![CircleCI](https://dl.circleci.com/status-badge/img/gh/drevops/behat-steps/tree/master.svg?style=shield)](https://dl.circleci.com/status-badge/redirect/gh/drevops/behat-steps/tree/master)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/drevops/behat-steps)
[![Total Downloads](https://poser.pugx.org/drevops/behat-steps/downloads)](https://packagist.org/packages/drevops/behat-steps)
![LICENSE](https://img.shields.io/github/license/drevops/behat-steps)

# Why traits?

Usually, such packages implement own Drupal driver with several contexts, 
service containers and a lot of other useful architectural structures.
But for this simple library, using traits helps to lower entry barrier for usage, 
maintenance and support. 
This package may later be refactored to use proper architecture. 

# Installation

`composer require --dev drevops/behat-steps`

**Heads up! Version `1.5.0` requires adding custom repositories to your project and allowing patching!**

Until upstream dependencies implement full support for Drupal 10, this project uses forked dependencies (maintained by DrevOps team). The follow-up release(s) of Behat Steps will have these reverted.

Add the following to your `composer.json`:
```json
{
    "require-dev": {
        "drevops/behat-steps": "^1.5",
        "cweagans/composer-patches": "^1.7",
        "drupal/drupal-extension": "dev-feature/drupal-10 as 4.2.1-dev"
    },
    "repositories": {
        "drupal/drupal-driver": {
            "type": "vcs",
            "url": "https://github.com/drevops/DrupalDriver"
        },
        "drupal/drupal-extension": {
            "type": "vcs",
            "url": "https://github.com/drevops/drupalextension"
        }
    },
    "prefer-stable": true,
    "minimum-stability": "dev",
    "extra": {
        "enable-patching": true
    },
    "config": {
        "allow-plugins": {
            "cweagans/composer-patches": true
        }
    }   
}
```

# Usage

Add required traits to `FeatureContext.php` ([example](tests/behat/bootstrap/FeatureContext.php)):

```
<?php

use Drupal\DrupalExtension\Context\DrupalContext;
use DrevOps\BehatSteps\ContentTrait;

/**
 * Defines application features from the specific context.
 */
class FeatureContext extends DrupalContext {

  use ContentTrait;

}
```

## Exceptions
- `\Exception` is thrown for all assertions.
- `\RuntimeException` is thrown for any unfulfilled requirements within a step. 

## Available steps

Use `behat -d l` to list all available step definitions.

There are also several pre and post scenario hooks that perform data alterations 
and cleanup. 

### Skipping before scenario hooks

Some traits provide `beforeScenario` hook implementations. These can be disabled
by adding `behat-steps-skip:METHOD_NAME` tag to your test. 

For example, to skip `beforeScenario` hook from `JsTrait`, add 
`@behat-steps-skip:jsBeforeScenarioInit` tag to the feature.

## Development

### Local environment setup

- Make sure that you have latest versions of all required software installed:
  - [Docker](https://www.docker.com/)
  - [Pygmy](https://github.com/pygmystack/pygmy)
  - [Ahoy](https://github.com/ahoy-cli/ahoy)
- Make sure that all local web development services are shut down (Apache/Nginx, Mysql, MAMP etc).
- Checkout project repository (in one of the [supported Docker directories](https://docs.docker.com/docker-for-mac/osxfs/#access-control)).  
- `pygmy up`
- `ahoy build` for Drupal 9 build or `DRUPAL_VERSION=7 ahoy build` for Drupal 7.
- Access built site at http://behat-steps.docker.amazee.io/  

To develop for another Drupal version, run `ahoy build` again.

Use `ahoy --help` to see the list of available commands.

### Apple M1 adjustments

Copy `default.docker-compose.override.yml` to `docker-compose.override.yml`.

### Behat tests

After every `ahoy build`, a new installation of Drupal is created in `build` directory.
This project uses fixture Drupal sites (sites with pre-defined configuration)
in order to simplify testing (i.e., the test does not create a content type
but rather uses a content type created from configuration during site installation).

- Run all tests: `ahoy test-bdd`
- Run all scenarios in specific feature file: `ahoy test-bdd path/to/file`
- Run all scenarios tagged with `@wip` tag: `ahoy test-bdd -- --tags=wip`
- Tests tagged with `@d7` or `@d9` will be run for Drupal 7 and Drupal 9 respectively.
- Tests tagged with `@d7` and `@d9` are agnostic to Drupal version and will run for all versions. 

To debug tests from CLI:
- `ahoy debug`
- Set breakpoint and run tests - your IDE will pickup incoming debug connection.

To update fixtures:
- Make required changes in the installed fixture site
- Run `ahoy drush cex -y` for Drupal 9 or `ahoy drush fua -y` for Drupal 7
- Run `ahoy update-fixtures` for Drupal 9 or `DRUPAL_VERSION=7 ahoy update-fixtures` for Drupal 7 to export configuration changes from build directory to the fixtures directory. 

---

For an end-to-end website DevOps setup, check out [DrevOps](https://drevops.com) - Build, Test, Deploy scripts for Drupal using Docker and CI/CD
