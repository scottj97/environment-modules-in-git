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

## Assumptions

* You have Environment Modules installed on your system

* You have a Unix user named `modules` that exists only for managing
  the modulefiles, and controlling updates to them.

* The path `/home/modules/modulefiles` is in your `modulesrc` file:

```
module use --append {/home/modules/modulefiles}
```

* There are no other MODULEPATH directories in use. (This is a
  limitation of `module unload localmodules`.)

* Other Unix users (not named `modules`) use the modules from
  `/home/modules/modulefiles` most of the time.

## Plan

Create a git repo for the modulefiles. Does this need to be separated
from `/home/modules/modulefiles`? Other users will be pushing to it so
probably yes...or use a branch?

Create a modulefile that, when loaded, switches MODULEPATH to a
locally-created git clone of the modulefiles. When unloaded, it
switches MODULEPATH back to the default.

After editing, testing, and commiting to the local clone, `git push`
updates the master repo, which (assuming the user knows the password
for user `modules`) automatically updates `/home/modules/modulefiles`.

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
```

### Manual clone

*This should be automated into a Modulefile!*

```
# As a regular user (i.e. anyone but user modules):
git clone /home/modules/modulefiles ~/modulefiles
cd ~/modulefiles
git remote set-url --push origin ssh://modules@localhost/home/modules/modulefiles
```

Then put `localmodules` into that module dir, and commit it.

## TBD

Update `localmodules` to do the git clone into a predefined directory
if that repo doesn't already exist. (Die if directory exists but is
not (the expected) git repo.)

Update that new Modulefile to warn user if their local clone is out of
date (needs `git pull`).
