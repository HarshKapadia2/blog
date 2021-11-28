---
title: "The .git Directory"
date: 2021-10-11T20:37:21+05:30
tags: ["git", "internals"]
author: ["Harsh Kapadia"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "The usecase of common files and directories in the '.git' directory of a Git repository."
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
editPost:
    URL: "https://github.com/HarshKapadia2/blog/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## The `.git` Directory

On executing the [`git init` command](https://harshkapadia2.github.io/git_basics/#_git_init) in a directory, Git creates a hidden `.git` directory in that directory. The `.git` directory contains all the project history data on which Git can perform its version control functions. It also contains files to configure the way Git handles things for that particular [repository](https://harshkapadia2.github.io/git_basics/#_repository).

## The `.git` Directory Contents

```
.git
├───addp-hunk-edit.diff
├───COMMIT_EDITMSG
├───config
├───description
├───FETCH_HEAD
├───HEAD
├───hooks
│   └───<*.sample>
├───index
├───info
│   └───exclude
├───lfs
│   ├───cache
│   │   └───locks
│   │       └───refs
│   │           └───heads
│   │               └───<branch_names>
│   │                   └───verifiable
│   ├───objects
│   │   └───<first_2_SHA-256_characters>
│   │       └───<next_2_SHA-256_characters>
│   │           └───<entire_64_character_SHA-256_hash>
│   └───tmp
├───logs
│   ├───HEAD
│   └───refs
│       ├───heads
│       │   └───<branch_names>
│       ├───remotes
│       │   └───<remote_aliases>
│       │       └───<branch_names>
│       └───stash
├───MERGE_HEAD
├───MERGE_MODE
├───MERGE_MSG
├───objects
│   ├───<first_2_SHA-1_characters>
│   │   └───<remaining_38_SHA-1_characters>
│   ├───info
│   └───pack
│       ├───<*.idx>
│       └───<*.pack>
├───ORIG_HEAD
├───packed-refs
├───rebase-merge
│   ├───git-rebase-todo
│   ├───git-rebase-todo.backup
│   ├───head-name
│   ├───interactive
│   ├───no-reschedule-failed-exec
│   ├───onto
│   └───orig-head
└───refs
    ├───heads
    │   └───<branch_names>
    ├───remotes
    │   └───<remote_aliases>
    │       └───<branch_names>
    ├───stash
    └───tags
        └───<tag_names>
```

## The `index` File

-   This file contains the details of [staged (added) files](https://harshkapadia2.github.io/git_basics/#_added_files) and is the staging area of the repository.

NOTE: The words 'index', 'stage' and 'cache' are the same in Git and are used interchangeably.

-   It is created when files are added for the first time and is updated every time the [`git add` command](https://harshkapadia2.github.io/git_basics/#_git_add) is executed.

![Index file explained](index-explained.png#center)

-   It is a binary file and just printing contents using `cat .git/index` will result in gibberish. Its contents can be accessed using the `git ls-files --stage` [plumbing command].

!['git ls-files --stage' command](git-ls-files.png#center)

-   From the image above

    -   `100644` is the mode of the file. It is an octal number.

        ```
        Octal: 10 0 644
        Binary: 001000 000 110100100
        ```

        -   The first six binary bits indicate the object type.
            -   `001000` indicates a regular file. (As seen in this case.)
            -   `001010` indicates a [symlink (symbolic link)](https://tdongsi.github.io/blog/2016/02/20/symlinks-in-git).
            -   `001110` indicates a [gitlink](https://www.oreilly.com/library/view/version-control-with/9780596158187/ch15s04.html#:~:text=gitlink).
        -   The next three binary bits (`000`) are unused.
        -   The last nine binary bits (`110100100`) indicate [Unix file permissions](https://harshkapadia2.github.io/cli/terminal.html#changing-permissions).
            -   `644` and `755` are valid for regular files.
            -   Symlinks and gitlinks have the value `0` in this field.

    -   The next 40 character hexadecimal string is the [SHA-1 hash](https://harshkapadia2.github.io/git_basics/#_sha1) of the file.
    -   The next number is a stage number/slot, which is useful during merge conflict handling.
        -   `0` indicates a normal un-conflicted file.
        -   `1` indicates the base, i.e., the original version of the file.
        -   `2` indicates the 'ours' version, i.e., the HEAD version with both changes.
        -   `3` indicates the 'theirs' version, i.e., the file with the incoming changes.
    -   The last string is the name of the file being referred to.

-   [More on the `index` file contents.](https://mincong.io/2018/04/28/git-index)
-   [More on the `index` file usage.](https://jwiegley.github.io/git-from-the-bottom-up/2-The-Index/1-meet-the-middle-man.html)

## The `HEAD` File

-   It is used to refer to the latest commit in the current branch.
-   Usually it does not contain a commit [SHA-1](https://harshkapadia2.github.io/git_basics/#_sha1), but contains the path to a file (of the name of the current branch) in the [`refs` directory](#the-refs-directory) which stores the last commit's SHA-1 hash in that branch.
-   It contains a commit's SHA-1 hash when [a specific commit or tag is checked out](https://harshkapadia2.github.io/git_basics/#_commits_sha). ([Detached `HEAD`](https://harshkapadia2.github.io/git_basics/#_detached_head) state.)
-   [More on the `HEAD` file.](https://harshkapadia2.github.io/git_basics/#_head)
-   Eg:

    ```shell
    # in the 'main' branch
    $ cat .git/HEAD
    ref: refs/heads/main
    $ git switch test_branch
    Switched to branch 'test_branch'
    $ cat .git/HEAD
    ref: refs/heads/test_branch
    ```

## The `refs` Directory

```
.git
├───...
└───refs
    ├───heads
    │   └───<branch_name(s)>
    ├───remotes
    │   └───<remote_alias(es)>
    │       └───<branch_name(s)>
    ├───stash
    └───tags
        └───<tag_name(s)>
```

-   This directory holds the reference to the latest commit in every local branch and fetched remote branch in the form of the SHA-1 hash of the commit.
-   It also stores the SHA-1 hash of the commit which has been [tagged].
-   The [`HEAD` file](#the-head-file) references a file (of the name of [the branch that is currently checked out](https://harshkapadia2.github.io/git_basics/#_new_branch_name)) from the `heads` directory in this (`refs`) directory.

## The `packed-refs` File

-   One file is created per branch and tag in the [`refs` directory](#the-refs-directory).
-   In a repository with a lot of branches and tags, there is a huge number of refs and a lot of the refs and tags are not actively used/changed.
-   These refs occupy a lot of storage space and cause performance issues.
-   The `git pack-refs` command is used to solve this problem. It stores all the refs in a single file called `packed-refs`.

![Print the packed-ref file](cat-packed-refs.png#center)

-   If a ref is missing from the usual `refs` directory after packing, it is looked up in this file and used if found.
-   Subsequent updates to a packed branch ref creates a new file in the `refs` directory as usual.

## The `logs` Directory

```
.git
├───...
└───logs
    ├───HEAD
    └───refs
        ├───heads
        │   └───<branch_name(s)>
        ├───remotes
        │   └───<remote_alias(es)>
        │       └───<branch_name(s)>
        └───stash
```

-   Contains the history of all commits in order.

![Print a branch's log file](cat-logs.png#center)

-   Every row consists of the parent commit's SHA-1 hash, the current commit's SHA-1 hash, the committer's name and e-mail, the [Unix Epoch Time](https://www.epochconverter.com/#:~:text=What%20is%20epoch%20time) of the commit, the time zone, the type of action and message in order.
-   There are logs for every branch in the local Git repository and for the fetched branches from the remote Git repository/repositories (if any).
-   Inside the `logs` directory
    -   The `HEAD` file stores information about all the commands executed by the user, such as branch switches, commits, rebases, etc.
    -   The files in the refs directory only include branch specific operations and history, such as commits, pulls, resets, rebases, etc.

## The `FETCH_HEAD` file

-   It contains the latest commits of the fetched remote branch(es).
-   It corresponds to the branch which was

    -   [Checked out](https://harshkapadia2.github.io/git_basics/#_new_branch_name) when last fetched.

        ![The contents of the FETCH_HEAD file](cat-FETCH_HEAD-1.png#center)

        -   From the image above, only one branch is displayed without the `not-for-merge` text. The odd one out (the 'main' branch in this case) is the branch which was checked out while fetching.

    -   Explicitly mentioned using the [`git fetch <remote_repo_alias> <branch_name>` command](https://harshkapadia2.github.io/git_basics/#_git_fetch).

        ![The contents of the FETCH_HEAD file](cat-FETCH_HEAD-2.png#center)

## The `COMMIT_EDITMSG` File

-   The commit message is written in this file.
-   This file is opened in an editor on executing the [`git commit` command](https://harshkapadia2.github.io/git_basics/#_git_commit).
-   It contains the output of the [`git status` command](https://harshkapadia2.github.io/git_basics/#_git_status) commented out using the `#` character.
-   If there has been a commit before, then this file will show the last commit message along with the `git status` output just before that commit.

## The `objects` Directory

```
.git
├───...
└───objects
    ├───<first_2_SHA-1_characters>
    │   └───<remaining_38_SHA-1_characters>
    ├───info
    └───pack
        ├───<*.idx>
        └───<*.pack>
```

-   The most important directory in the `.git` directory.
-   It houses the data (SHA-1 hashes) of all the [commit, tree and blob objects](https://harshkapadia2.github.io/git_basics/#_git_objects) in the repository.
-   To decrease access time, objects are placed in buckets (directories), with the first two characters of their SHA-1 hash as the name of the bucket. The remaining 38 characters are used to name the object's file.
-   [More on the `pack` directory.](https://harshkapadia2.github.io/git_basics/#_the_pack_directory)

## The `info` Directory

```
.git
├───...
└───info
    └───exclude
```

-   It contains the `exclude` file which behaves like the [`.gitignore` file](https://harshkapadia2.github.io/git_basics/#_gitignore_file), but is used to ignore files locally without modifying `.gitignore`.
-   [More on the `exclude` file.](https://harshkapadia2.github.io/git_basics/#_ignore_files_locally_without_modifying_gitignore)

## The `config` File

-   This file contains the local Git repository configuration.
-   It can be modified using the [`git config --local` command](https://harshkapadia2.github.io/git_basics/#_git_config).

## The `addp-hunk-edit.diff` File

-   Created when the `e` (edit) option is chosen in the [`git add --patch` command](https://harshkapadia2.github.io/git_basics/#_p_or_patch).
-   Enables the manual edit of a hunk of a file to be staged.

## The `ORIG_HEAD` File

-   It contains the SHA-1 hash of a commit.
-   It is the previous state of the HEAD, but not necessarily the immediate previous state.
-   It is set by certain commands which have destructive/dangerous behaviour, so it usually points to the latest commit with a destructive change.
-   It is less useful now because of the [`git reflog` command] which makes reverting/resetting to a particular commit easier.

## The `description` File

-   This is the description of the repository.
-   This file is used by [GitWeb](https://git-scm.com/book/en/v2/Git-on-the-Server-GitWeb), which hardly anyone uses today, so can be left alone.
