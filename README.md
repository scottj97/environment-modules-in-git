# Environment Modules in Git

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

* You have Environment Modules installed on your system

* You have a Unix user named `modules` that exists only for managing
  the modulefiles, and controlling updates to them.

* The path `/home/modules/modulefiles` is in your `modulesrc` file:

```
module use --append {/home/modules/modulefiles}
```

* There are no other MODULEPATH directories in use. (This is a
  limitation of my `localmodules` unload process.)

* Other Unix users (not named `modules`) use the modules from
  `/home/modules/modulefiles`.

## Principles of Operation

First we create a git repo for the modulefiles.

Then we install a modulefile that, when loaded, switches MODULEPATH to
a locally-created git clone of the modulefiles. When unloaded, it
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
git commit -m 'Initial checkin of existing modulefiles'

# Enable updates when receiving pushes:
git config --local receive.denyCurrentBranch updateInstead

# Get localmodules file from GitHub
curl --output localmodules https://raw.githubusercontent.com/scottj97/environment-modules-in-git/master/localmodules

# Edit paths in the top of localmodules, if your installation differs from the assumptions, then:
git commit -am 'Add localmodules from github.com/scottj97/environment-modules-in-git'

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
