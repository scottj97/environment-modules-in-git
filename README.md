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

* Other Unix users (not named `modules`) use the modules from
  `/home/modules/modulefiles` most of the time.

## Plan

Create a git repo for the modulefiles. Does this need to be separated
from `/home/modules/modulefiles`? Other users will be pushing to it so
probably yes...or use a branch?

Create a modulefile that, when loaded, switches MODULEPATH to a
locally-created git clone of the modulefiles. When unloaded, it
switches MODULEPATH back to the default.

After editing and commiting to the local clone, `git push` updates the
master repo. Can this automatically update `/home/modules/modulefiles`
(assuming the user knows the password for user `modules`)? Or must the
user go manually do a `git pull` in `/home/modules/modulefiles` as
user `modules`?

## Execution

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

## TBD

Need to clone via filesystem, but push via SSH (so users must enter
password for user `modules`).

Need to write Modulefile to switch to local repo when loaded.

Update that new Modulefile to do the git clone into a fixed directory
if that repo doesn't already exist.

Update that new Modulefile to warn user if their local clone is out of
date (needs `git pull`).
