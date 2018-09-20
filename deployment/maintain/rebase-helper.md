---
title: Rebase helper
subsection: maintain
order: 2
---

# rebase-helper
rebase-helper is a tool which helps package maintainers with updating package to the latest upstream version.
It automates a lot of manual tasks the package maintainer usually does, when a new upstream version of a package is released.

## Install rebase-helper
Begin installation on Fedora using the ``dnf`` command:

```sh
$ sudo dnf install rebase-helper
```

It requires several other programs like ``abipkgdiff``, ``rpmdiff``, ``mock``, ``fedpkg``, ``meld``, etc.
These programs are installed automatically as dependencies of rebase-helper.

Note: rebase-helper is also available as EPEL-7 package. Feel free to use it on CentOS and RHEL systems.


## What do I get from it?

**rebase-helper** always creates *rebase-helper-results* directory containing the following items:

| Path                  | Description                                                       |
|:--------------------- |:----------------------------------------------------------------- |
| *report.txt*          | summary report with all important information                     |
| *changes.patch*       | diff against original files, directly applicable to dist-git repo |
| *logs/*               | log files of various verbosity levels                             |
| *rebased-sources/*    | git repository with all modified files                            |
| *checkers/*           | reports from individual checkers that were run                    |
| *old-build/*          | logs and results of old (original) version build                  |
| *new-build/*          | logs and results of new (rebased) version build                   |

## How does it work?

The following steps describe a rebase process:

- **Preparation**

    - *rebase-helper-workspace* and *rebase-helper-results* directories are created
    - original SPEC file is copied to *rebase-helper-results/rebased-sources* directory and its Version tag is modified


- **Getting sources**

    - old and new source tarballs are downloaded and extracted to *rebase-helper-workspace* directory
    - old sources are downloaded from lookaside cache if possible


- **Downstream patches**

    - new git repository is initialized and the old sources are extracted and commited
    - each downstream patch is applied and changes introduced by it are commited
    - new sources are extracted and added as a remote repository
    - `git-rebase` is used to rebase the commits on top of new sources
    - original patches are modified/deleted accordingly
    - resulting files are stored in *rebase-helper-results/rebased-sources*
    - diff against original files is saved to *rebase-helper-results/changes.patch*


- **Build**

    - old and new source RPMs are created and built with selected build tool
    - old SRPM and RPMs can also be downloaded from Koji to speed up the rebase


- **Comparison**

    - multiple checker tools are run against both sets of packages and their output is stored in *rebase-helper-results/checkers* directory


- **Cleanup**

    - *rebase-helper-workspace* directory is removed

## How to rebase your packages?
Let's say we want to rebase a package foobar from ``foobar-1.2.0`` to ``foobar-1.2.1`` using rebase-helper:

```sh
# Change to the location of foobar.spec and other package components (cloned dist-git dir), e.g.
$ cd $HOME/rpmbuild/REPOS/foobar

# Update to the selected upstream version
$ rebase-helper 1.2.1

# Alternatively, you can run **rebase-helper** in a container:
$ docker run -it -e PACKAGE=foo rebasehelper/rebase-helper:latest
```

Alternatively, if the package has been added to one of the release monitoring services, it is not needed to use the version and simply call rebase-helper with no arguments and it will figure out, what latest version is available.

```sh
# rebase-helper invocation with immediate application of the changes to the local repository as a commit,
# if the rebase succeeded.
rebase-helper --apply-changes

# Alternatively run rebase-helper with --update-sources parameter to have the latest sources uploaded
# to dist-git, as well as apply the produced 'changes.patch' as a commit to the local repository
rebase-helper --apply-changes --update-sources
```

If you do not want to be bothered, add the ``--non-interactive`` option to rebase-helper's command line 

See https://rebase-helper.readthedocs.io/en/latest/user_guide/usage.html for complete explanation of all possible `rebase helper` arguments

After rebase-helper finishes, check the output.

