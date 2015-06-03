# drush-config-workflow
Jumpstart your Drupal configuration merge magic with this repo's code and instructions. 

[Git Workflow](docs/git_workflow.md)       | [Rsync Workflow](docs/rsync_workflow.md)   | [Three-way merge](docs/three_way_merge.md)
------------------------------------------ | ------------------------------------------ | ------------------------------------------
[![Git workflow](docs/img/git_workflow.png)](docs/git_workflow.md) | [![Rsync workflow](docs/img/rsync_workflow.png)](docs/rsync_workflow.md) | [![Three-way merge](docs/img/kdiff3-user-field-conflicts.png)](docs/three_way_merge.md)


## Overview
This repository provides code and instructions to help you get started on setting up a configuration workflow for your Drupal 8 project.  The Drush `config-merge` command is a powerful tool to use in instances where different team members need to change the site's configuration settings at the same time.  With `config-merge`, you can combine configuration changes made on two different instances of the same Drupal site.  When the configuration changes are not conflicting, the merge happens automatically, after a confirmation message listing the sections changed is presented.  If configuration changes _are_ in conflict, then a three-way merge tool will be launched, allowing for the conflicts to be reconciled by hand in the configuration YAML files.

## Installation
To get started, please read the [INSTALL](INSTALL.md) document.  It includes instructions on how to run a quickstart script that will set everything up for you with minimal effort. Manual installation instructions are also provided for those who do not wish to run the script.

## Using Config Merge
Config merge always operates by merging the configuration changes from a "remote" Drupal site into the current configuration settings of a local site. When the operation is complete, then the local site will have all of the configuration changes from both systems; the remote system remains unmodified.  It is advisable to deploy the local site to the remote site after running config-merge, so that the two sites can remain in sync.

There are two modes to use `config-merge` in: `git mode`, and `rsync mode`.  The mode determines how the `config-merge` command will move changes from the remote system to the local system; regardless of the mode, it is always required that the configuration files on the local site be committed to a git repository, as `config-merge` uses git to merge the configuration changes.

### Git Mode

The `config-merge` command can be used in 'git mode' as follows:
```
drush config-merge @remote --git
```
In `git mode`, config-merge will use ssh to export the current configuration changes on the remote site, commit them on a branch, and then push the branch to the central repository.  This branch is then pulled to the local machine, and merged in with the active configuration of the local site.  The advantage of `git mode` is that it allows git to track the point where each branch diverged, allowing for accurate merges; however, `git mode` requires ssh + git access on the remote site, and that is not always available.

Please read the [git configuration workflow documentation](docs/git_workflow.md) for more information on how this works.

### Rsync Mode

The `config-merge` command can be used in 'rsync mode' as follows:
```
drush config-merge @remote --temp
```
In `rsync mode`, config-merge will first export the currnet configuration changes on the remote site, and then use rsync to copy the files back to the local system, where they are committed to a temporary branch and then merged in with the active configuration of the local site.  The advantage of `rsync mode` is that it works in managed environments such as Pantheon, where it is not possible to run remote git commands via ssh.  In `rsync mode`, however, it is necessary for you to keep track of the base commit that the remote side was deployed from.  `config-merge` can erase local configuration changes if ran in `rsync mode` with the wrong parameters.

Please read the [rsync configuration workflow](docs/rsync_workflow.md) for more information on how this works.

