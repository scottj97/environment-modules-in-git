# Environment Modules in Git

## Obsolete

This project has been moved to the
[Cookbook](https://modules.readthedocs.io/en/stable/cookbook/modulefiles-in-git.html)
in the official project.

## What?

This is a demonstration of how to version control the Modulefiles
from [Environment Modules](http://modules.sourceforge.net/) using
[Git](https://git-scm.com/).

The Modules concept is a great way to manage versions of tools on a
common server, but managing the Modulefiles themselves is error-prone
when not controlled using a SCM like Git.

## Goals

* To be able to create, edit, and test Modulefiles without the risk of
  breaking other users.

* To be able to track changes to the system-wide Modulefiles using
  Git.

* To enable testing of new tool versions (with their associated
  Modulefiles) before making the new version generally available
  (since the new Modulefile will not be public until pushed).

## Assumptions

* You have Environment Modules version 4.2.1 installed on your system,
  and Git version 2.4 or later.

* You have a Unix user named `modules` that exists only for managing
  the Modulefiles, and controlling updates to them.

* The path `/home/modules/modulefiles` is in your `modulerc` file:

```
module use --append {/home/modules/modulefiles}
```

* Other Unix users (not named `modules`) use the modules from
  `/home/modules/modulefiles`.

## Principles of Operation

First we create a git repo for the Modulefiles.

Then we install a modulefile that, when loaded, switches MODULEPATH to
a locally-created git clone of the Modulefiles. When unloaded, it
switches MODULEPATH back to the default.

After this, any time a user wants to edit the Modulefiles, he works in
his local git repo. After editing, testing, and commiting to the local
git repo, `git push` updates the master repo, which (assuming the user
knows the password for user `modules`) automatically updates
`/home/modules/modulefiles`.

## Execution

### Setup

Set up the master repo at `/home/modules/modulefiles`, as user `modules`:
```
# Convert the existing Modulefiles into a git repo:
cd /home/modules/modulefiles
git init
git add .
git commit -m 'Initial checkin of existing Modulefiles'

# Enable updates when receiving pushes:
git config --local receive.denyCurrentBranch updateInstead

# Edit your global $MODULESHOME/init/modulerc file to use the new env
# var MODULES_REPO instead of hard-coding the path (requires Modules
# 4.1). Your modulerc should have:
##   setenv MODULES_REPO /home/modules/modulefiles
##   module use /\$MODULES_REPO
# The extra slash required before \$MODULES_REPO is a bug: https://github.com/cea-hpc/modules/issues/223

# Get localmodules file from GitHub
curl --output localmodules https://raw.githubusercontent.com/scottj97/environment-modules-in-git/master/localmodules

# Edit paths in the top of localmodules, if your installation differs from the assumptions, then:
git add localmodules
git commit -m 'Add localmodules from github.com/scottj97/environment-modules-in-git'

```

### Usage

```
# As a regular user (i.e. anyone but user modules):
module load localmodules
cd $HOME/modulefiles

# Edit, test, then:
git commit -am 'Make some edits'
git push

module unload localmodules
```

After a successful push and unload, it is safe to delete your local
`$HOME/modulefiles` directory if you wish.
