# Git Flow Presentation

This directory contains files for use demonstrating Git Flow in action with the [BU Navigation plugin](https://github.com/bu-ist/bu-navigation).

### Goals

* To demonstrate the use of "hotfix" branches by fixing a bug with navigation links.
* To demonstrate the use of "feature" branches by adding support for "Private" posts in administrative navigation views.

Both have already been implemented in the official repository. This directory contains modified versions of the plugin repository which have been `git reset --hard` to earlier commits for the purpose of this presentation.

There are two directories -- `plugins-before` and `plugins-after`. The former contains the starting point, the latter contains the end result. The procedures that take each from before to after are detailed below.

### Plugin Versions

* bu-navigation-hotfix -- A version of the plugin in need of a hotfix for a bug that is causing external links to be excluded from navigation links.
* bu-navigation-feature -- A version of the plugin prior to the additon of private post type support in administrative navigation views.
* bu-navigation-feature-with-merge-conflicts -- Same as above, but with a different branch point for feature/private-posts that introduces a merge conflict upon reintegration with develop.

### Patches

The `patches` directory adjacent to this README contains three separate patch files that can be applied to simulate development of each feature / hotfix. The patches are named after the plugin directory they are intended to be applied to.

## Hot Fix

* **Directory**: plugins-before/bu-navigation-hotfix
* **Patch**: patches/bu-navigation-hotfix.patch
* **Starting commit**: 1.1 (master)

### Background

Right after the initial open source release a bug was discovered — external navigation links were not showing up in navigation lists.

We know that this bug did not exist in the last internal release, which means that the bug was introduced as a side effect (regression) during recent development.

We will use `git bisect` to find the commit that introduced the bug and revert.

### Setup

* A theme with `add_theme_support( "bu-navigation-primary" )` that uses `bu_navigation_display_primary` for primary nav bar
* A few published top-level pages
* A top-level navigation link

### Steps

```bash
$ git checkout master   # Current release (has bug)
$ git bisect start
$ git bisect bad
$ git checkout 1.0.1    # Last known release without the bug
$ git bisect good
```

Walk through the bisect process while refreshing the page
Show the refs/bisect tags in gitx

```bash
$ git bisect reset      # Removes refs/bisect/*

# The last command should return to the commit you were on when `git bisect start` was issued
# If not...
$ git checkout master

# Create our hotfix branch and fix the bug
$ git checkout -b hotfix/missing-nav-links
$ patch -p0 < ../../bu-navigation-hotfix.patch

# Note that this patch includes both the fix and version bumps / changelog

$ git commit -am “Fix bug in exclusion logic introduced in 200366ca42

Was causing external navigation links to be hidden in navigation lists.”

# Merge to production, deploy ASAP
$ git checkout master
$ git merge hotfix/missing-nav-links —-no-ff
$ git tag -a 1.1.1

# Merge back in to develop branch
$ git checkout develop
$ git merge hotfix/missing-nav-links —-no-ff
```

Next step?  Add unit test coverage.

## Feature

* **Directory**: plugins-before/bu-navigation-feature
* **Patch**: patches/bu-navigation-feature.patch
* **Starting commit**: 145e6059a3 (develop)

### Background

The initial release only supported display of published content in the administrative navigation views.
One of the first enhancement requests was to include “Private” pages so that folks could stage sections of private content prior to publishing.

### Setup

* A few published top-level pages
* A few privately published pages

### Steps

```bash
$ git checkout develop

# Create the feature branch and implement the feature
$ git checkout -b feature/private-posts
$ patch -p 0 < ../../patches/bu-navigation-feature.patch
$ git commit -am "Adding support from 'Private' posts in admin navigation views"

# Merge feature branch -> develop
$ git checkout develop
$ git merge feature/private-posts

# Explain what “fast-forward means”, and why we want to avoid it

# Undo the previous commit
$ git reset --hard HEAD@{1}     # or `git reset --hard “HEAD^"` <-- zsh requires quotes due to ^

# Merge again, this time with a merge commit
$ git merge feature/private-posts --no-ff

# Delete the reintegrated branch
$ git branch -d feature/private-posts     # Delete after re-integration, no longer needed
```

## Feature w/ Merge Conflict

* **Directory**: plugins-before/bu-navigation-feature-merge-conflicts
* **Patch**: patches/bu-navigation-feature-merge-conflicts.patch
* **Starting commit**: 24203d5e77

### Background

The initial release only supported display of published content in the administrative navigation views.
One of the first enhancement requests was to include “Private” pages so that folks could stage sections of private content prior to publishing.

### Setup

* A few published top-level pages
* A few privately published pages

### Steps

```bash
$ git checkout 24203d5e77
$ git checkout -b feature/private-posts
$ patch -p 0 < ../bu-navigation-feature-merge-conflicts.patch
$ git commit -am "Adding support from 'Private' posts in admin navigation views"
$ git checkout develop
$ git merge feature/private-posts --no-ff

# Conflict! Talk about what causes them, and our approach to resolving them.
# Mention git checkout --ours and git checkout --theirs for subversion folks
# Manually regenerate bu-navigation.js from bu-navigation.dev.js using http://closure-compiler.appspot.com/home

$ git status
$ git add js/bu-navigation.js
$ git commit

$ git branch -d feature/private-posts
```
