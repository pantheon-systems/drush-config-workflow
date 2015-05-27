# Git Workflow

![git workflow](img/git_workflow.png)

The diagram above shows the git workflow supported by Drush `config-merge`.  The git workflow is the best mode to use when you have full ssh access to the remote site, and can run git commands over ssh.  If you are on Pantheon, or are using some other system where this is not possible, then you should use the [rsync workflow](rsync_workflow.md).

## Getting Started

It is recommended that you set up two local sites as described in the [INSTALL](../INSTALL.md) document for experimentation purposes.  You may also use an existing site that you provide, if you wish.

## Example Scenario

Alice and Bob used to work together at a Cryptography company, but now they design websites at a small web shop they started.  Alice does all of the back-end coding and module development, while Bob handles the theming and site building.  They often collaborate on tasks and share ideas using a staging server that Alice set up.

When Alice is working on a new feature, she commits her code and configuration changes to a feature branch in their git repository.  To implement her feature, she configured a new content type, and created a custom module with a form alter method.  She uses the drush `config-export` command to create commit (F1) in the diagram, first adding her custom code changes with `git add`, so that they are included in the same commit as the configuration they depend on.
```
$ git checkout -b my-perfect-feature
$ git add modules/custom
$ drush @dev config-export --commit --message="Code and configuration for my feature."
```
Once Alice’s new feature really is perfect, she merges it in to the master branch, and prepares tag <T1>.  It is important to create a tag for every deployment, so that it is possible to track what was pushed to the staging server.  Usually, tags are version strings following the conventions of semantic versioning.
```
$ git checkout master
$ git merge my-perfect-feature
$ git push origin master
$ git tag T1
$ git push origin T1
```
Alice can deploy her changes on the staging server directly from her dev machine, if she wants.
```
$ drush @stage ssh git fetch
$ drush @stage ssh git checkout T1
$ drush @stage updatedb
$ drush @stage config-import
```
Bob is now ready to go to work.  He has defined some blocks and set up the basic structure of the site, and saved them in the database on the staging server.

Alice has not been idle, though; she has made changes to her content type, and committed them to the repository as commit (F2).  When Alice sees that Bob has made some progress with the site structure, she decides to merge the changes in to her new feature branch.  Alice is going to use the Drush `config-merge` command to do this.
```
$ drush @dev config-merge --git --message="Merge in Bob’s work."
```
This command does quite a lot.  The --git flag tells `config-merge` that Alice would like to use git push and git pull to move the changes from the staging machine to her machine.  Alice has set up the staging server such that it can push to the central repository, so this mode is available for use.  In “git” mode, the config-export command does the following operations:

1. It does a config-export on @stage, and commits the configuration as (C1)
1. It also does a config-export on @dev, committing the change as (F3)
1. It pushes C1 from @stage to the central git repository, and pulls it on @dev so that it is available to be merged.
1. It runs git merge, creating commit (M1)
1. Finally, it runs config-import on @dev, making Bob’s configuration changes active on Alice’s dev machine.

At this point in development, none of Alice or Bob’s changes overlap, so the config-merge command does this all automatically.

Alice then merges her feature branch back into the master branch as commit (2), but she then discovers that Bob has made more changes to the staging server while she was merging.  If Alice deployed now, his additional changes [c] would be overwritten by the config-import, so Alice waits until he is done, making some additional configuration changes herself while she is waiting.

Alice then runs the config-merge command again, to combine configuration changes [c] and [w].  This time, Alice discovers that she and Bob both changed the same content type.  When config-merge runs git merge, this conflict will be flagged, and a three-way diff tool will be brought up, and Alice will have the opportunity to review and resolve the conflicts.

[kdiff3 diagram--if I am going to show a diff, then I need to have an example of what is being changed for context.]

Once this is done, Alice creates tag <T2> and does another deployment to the staging server, bringing Bob’s environment back into sync with her changes.
