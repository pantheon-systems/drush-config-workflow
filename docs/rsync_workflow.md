# Rsync Workflow

![rsync workflow](img/rsync_workflow.png)

The diagram above shows the rsync workflow supported by Drush `config-merge`.  The rsync workflow is the best mode to use when working with Pantheon, or other systems where it is not possible to run git operations over ssh.  If you have full ssh shell access to the remote site, than you should consider using the  [git workflow](git_workflow.md) instead.

## Getting Started

It is recommended that you set up a free Pantheon development site to use for experimentation purposes, and then make a local copy of it as described in the [INSTALL](../INSTALL.md) document.  You may also use an existing site that you provide, if you wish.

