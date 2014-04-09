git-house-guidelines
====================

My git house guidelines for Drupal projects I work on.

## General stuff, TLDR ##

* Keep commits small and separate.
* Write clean log messages.
* Don't hack core!

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

If there are lots of patches on the issue, specify the link to the actual comment:

* 'Applied patch from http://drupal.org/node/1234567#comment-7654321: Fixed evil bug that was causing evil.'

(Note that since drupal.org upgraded to Drupal 7, the permalinks on comments do not give the node ID, and should thus NOT be used. Use the link in the 'Files' table in the issue summary instead.)

This considerably simplifies the task of checking for divergence before updating a module to a newer version, and allows use of the following workflow.

#### Contrib module update workflow ####

The following will give a clear list of all updates and patches applied to a module:

```
git log sites/all/modules/contrib/foobar
```

There are two possibilities:

- The most recent commit is an update or the initial import, in which case everything is fine and you can go ahead and make the new update.
- There are patch commits before the latest update commit or the initial import.

In the second case, you will need to take the following steps:

1. Check all the issues for the patches to see whether they are included in the new release. Make a list of the commits whose patches have not been included.
2. Do ```drush up MODULE``` as normal, and make a commit for the updated module code.
3. Now re-patch the work through the list of commits. The simplest thing to try first is to cherry-pick the patch commit:
    
    ```git cherry-pick SHA```
4. If that doesn't work, you will need to reroll the patch (and then re-upload it to the issue on drupal.org, and then make a new commit specifying the comment which has your new patch).

## Branches ##

### Feature branches ###

Feature branches may be used for a single new feature that requires a lot of commits that don't make sense in isolation, such as a contrib module that requires heavy patching, or changes to Features and their knock-on effects. 

These should branch off dev and merge back into it (with the --no-ff option so that the branch history is preserved). Dev is then merged into master.

The naming convention for a feature branch is 'feature-ISSUE#-description', where ISSUE is the id number of the issue in the project's issue tracker (assuming there is a tracker and an issue number). Alternatively, use a date in the name, thus: 'feature-YYYY-MM-DD-description'.

* To create a feature branch:

    ```git checkout dev```
    
    ```git checkout -b FEATUREBRANCH```
* If dev is updated in the meantime, and you want those changes available to the feature branch:

    ```git merge dev```
    
    (If you haven't yet pushed the branch at all, you can rebase on dev instead, which produces cleaner history.)
* To merge into dev when the feature is complete, there are three possible techniques:
    * Merge the feature branch in, preserving it in the history as a separate set of commits:

        ```git checkout dev```
        
        ```git merge --no-ff FEATUREBRANCH```
    
        This will usually be the preferred option, and will always be available.
    
    * If the dev branch has has no new commits since the feature branch forked from it, you can allow git to fast-forward the merge. This results in a linear history, as it the commits had been on dev all along:
    
        ```git checkout dev```
        
        ```git merge FEATUREBRANCH```
    
    * The work on the feature branch can be applied as a single commit to the dev branch, using:
    
        ```git merge --squash dev```
        
        This means that the dev branch has none of the history of changes and log messages from the feature branch. 
    
        This technique is especially useful when working on local feature branches that you never push to the repository, as it allows you to do rough work on the local branch, and the collapse it all into a single commit for public consumption.
    
### Hotfix branches ###

Once a project has launched, make a hotfix branch for urgent fixes. These branch off master, and are merged back into *both* master and dev. The naming convention for a hotfix branch is 'hotfix-ISSUE#-description'.

To create a hotfix branch:

```git checkout master```

```git checkout -b HOTFIXBRANCH```

Once the hotfix is ready to deploy, merge it back into both dev and master:

```git checkout master```

```git merge --no-ff HOTFIXBRANCH```

```git checkout dev```

```git merge --no-ff HOTFIXBRANCH```

Any non-urgent clean-up can now be done on the dev branch, which can be merged into master as usual.

