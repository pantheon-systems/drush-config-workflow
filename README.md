# drush-config-merge
Jumpstart your Drupal configuration merge magic with this repo's code and instructions. 

## Overview
The Drush `config-merge` command is a powerful tool to use in your configuration workflow.
With this command, you can combine configuration changes made on two Drupal sites.  When the
configuration changes are not conflicting, then the merge happens automatically, after a 
confirmation message listing the sections changed is presented.  If configuration changes
_are_ in conflict, then a three-way merge tool will be launched, allowing for the conflicts
to be reconciled by hand in the configuration YAML files.

## Quickstart

To quickly create two local sites for `config-merge` testing, run:

./scripts/quickstart

This will install the requirements and set up an alias file for testing.
See the sections below for details.

## Requirements
* A config-merge tool, e.g. `kdiff3`, must be installed
* Drush 7.0.0-rc2 (only), or Drush 8.x-dev
* Pantheon Terminus CLI (Optional -- only needed if using a Pantheon remote)
* Drupal 8.0-beta10 or later
* A local Drupal "dev" site, and a local or remote "staging" site

### Installing kdiff3

* **Mac:** brew install Caskroom/cask/kdiff3
* **Linux:** sudo apt-get install kdiff3
* **Windows:** When using Windows, it is recommended to use Drush and Drupal inside of a virtualized environment, such as [kalabox](http://www.kalamuna.com/products/kalabox/) or [Varying Drupal Vagrants](https://github.com/gman29/varying-drupal-vagrants/blob/master/www/vvv-hosts).

### Installing Drush 7 or 8
Follow the [installation instructions in the Drush documentation](http://docs.drush.org/en/master/install/) to install Drush 7.  Most users should be able to upgrade from Drush 5 or Drush 6 to version 7 without encountering any issues.  At the time of this writing, Drush 7.0.0-rc2 is recommended.  However, once Drush 8.0-alpha1 is released, this version should be used with Drupal 8, as at the moment, only this version of Drupal is supported by the config-merge command.  A Drupal 7 version may be provided in the future.

### Installing Terminus CLA

Follow the [terminus installation instructions](https://github.com/pantheon-systems/cli/wiki/Installation).

### Installing Drupal 8
If you already have a remote Drupal 8 testing site, then you may use it to try out 
config-merge.  Otherwise, you'll need to make a test site.

* **Remote, on Pantheon:** [Spin up a remote Drupal 8 environment](https://dashboard.pantheon.io/products/drupal8/spinup)
* **Local, with Drush:** `drush qd sitename --core=drupal-8.0.x --db-url='mysql://root@localhost:/dbname'`

Adjust your database credentials as necessary, if using Drush.

### Create a local "dev" site
tbd
