git-house-guidelines
====================

My git house guidelines for Drupal projects I work on.

## General stuff, TLDR ##

* Keep commits small and separate.
* Write clean log messages.

## Commit granularity ##

In general, keep commits small and atomic.

* Keep core, contrib, and custom code in separate commits (custom code includes custom modules, features, and themes: with these it's a little less clear-cut when to split and when not to).
* One commit per contrib module import, unless it's a bunch of closely related ones.
* One commit per contrib module update, so we can easily back out of any problem updates.
* One commit per contrib module patch, so we can easily keep track of divergent contrib modules when updating.
* One commit per change to a feature, but accompanying changes to custom module core or theming are ok to be included with it. (In other words, an atomic change to site functionality: if you add a field in a feature, you may also include the change in the theme's CSS file that styles it. However, it's just as fine to have these as two commits, as will be the case if these are done by different people.)

## Commit messages ##

Follow the Drupal commit message format (see http://drupal.org/node/52287) with regard to starting the message with a past tense verb, thus:

* 'Fixed evil bug that was causing evil.'
* 'Added cool feature.'
* 'Changed the snark to a boojum.'

Special cases are detailed below.

### Importing and updating contrib modules ###

Suggested format is based on what the 'drush up' command outputs:

* 'Imported foobar (7.x-1.1).'
* 'Updated to foobar-7.x-1.2.'

(The difference in format here is because of drush; it seems simplest to just copy and paste that output.)

### Patching contrib modules

Any patch applied in our repository to a contrib module should also be uploaded to drupal.org. The commit message for a patch to a contrib module therefore refers to the issue node on drupal.org:

* 'Applied patch from http://drupal.org/node/1234567: Fixed evil bug that was causing evil.'

If there are lots of patches, it may be a good idea to specify the link to the actual comment:

* 'Applied patch from http://drupal.org/node/1234567#comment-7654321: Fixed evil bug that was causing evil.'

The considerably simplifies the task of checking for divergence before updating a module to a newer version. The following will give a clear list of all updates and patches applied to a module:

```
git log sites/all/modules/contrib/foobar
```

There are two possibilities:

- The first commit is an update or the initial import, in which case everything is fine and you can go ahead and make the new update.
- There are patch commits before the latest update commit or the initial import.

In the second case, you will need to check all the issues for the patches to see whether they are included in the new release. For patches which are not included, you can try to cherry-pick the patch commit after an update:

```
git cherry-pick SHA
```

If that doesn't work, you will need to reroll the patch (and then re-upload it to the issue on drupal.org, and then make a new commit specifying the comment which has your new patch).

## Branches ##

### Hotfix and feature branches ###

Feature branches may be used for a single new feature that requires a lot of commits that don't make sense in isolation, such as a contrib module that requires heavy patching. These should branch off dev and merge back into it (with the --no-ff option so that the branch history is preserved). Dev is then merged into master. The naming convention for a feature branch is 'feature-ISSUE#-description', where ISSUE is the id number of the issue in the project's issue tracker (assuming there is a tracker and an issue number).

You can of course create local feature branches that you never push to the repository by using 'git merge --squash'. This collapses all of your commits on the local branch to a single large commit.

Once a project has launched, make a hotfix branch for urgent fixes. These branch off master, and are merged back into *both* master and dev. The naming convention for a hotfix branch is 'hotfix-ISSUE#-description'.

