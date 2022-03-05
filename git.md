Branches are simply pointers to a specific commit -- nothing more.

# Chapter 1. Getting Started

## 1.3 What is Git?

### 1.3.1 The working tree, staging area and Git directory

There are three main sections of a Git project: the working tree, the staging area, and the Git directory.

- The ***working tree(directory)*** is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.
- The ***staging area***, or ***index***, is a file, generally contained in your Git directory, that stores information about what will go into your next commit.
- The Git directory, is where Git stores the metadata and object database for your project. It's what is copied when you clone a repository from another computer.

### 1.3.2 The Three States

Git has three main states that your files can reside in: *modified*,*staged*, and *committed*.

- If a particular version of a file is in the Git directory, it's consider ***committed***. 
- If it has been modified and was added to the stagging area, it's ***staged***.
- And if it was changed since it was checked out but has not been staged, it's ***modified***.

## 1.6 First-Time Git Setup

You may need to do a few things to customize your Git environment after you newly installed Git on your system.

Git comes with a tool called `git config` that lets you get and set configuration variables that control all aspects of how Git looks and operates. These variables can be stored in three different places:

1. `/etc/gitconfig` file: Contains values applied to every user on the system and all their repositories. If you pass the option `--system` to `git config`, it reads and writes from this file specifically.
2. `~/.gitconfig` or `~/.config/git/config` file: Values specific personally to you, the user. You can make Git read and write to this file specifically by passing the `--global` option, and this affects all of the repositories you work with on your system.
3. `config` file in the Git directory (`.git/config`): located in your currently used repository directory. You can force Git to read from and write to this file with the `--local` option, but that is in fact the default.

Each level overrides values in the previous level, so values in `.git/config` trump those in `/etc/gitconfig`.

---

**Your Identity**

The first thing you should do when you install Git is to set your user name and email address. This is important because every Git commit uses this information, and it’s immutably baked into the commits you start creating:

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

**Your Editor**

If you want to use a different text editor, such as Emacs, you can do the following:

```
git config --global core.editor emacs
```

**Your default branch name**

By default Git will create a branch called master when you create a new repository with `git init`.

To set `main` as the default branch name do:

```
git config --global init.defaultBranch main
```

**Checking Your Settings**

```
$ git config --list
user.name=John Doe
user.email=johndoe@example.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...

$ git config user.name
John Doe

$ git config --show-origin rerere.autoUpdate
file:/home/johndoe/.gitconfig
false
```

## 1.7 Getting Help

If you ever need help while using Git, there are three equivalent ways to get the comprehensive manual page (manpage) help for any of the Git commands:

```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

In addition, if you don’t need the full-blown manpage help, but just need a quick refresher on the available options for a Git command, you can ask for the more concise “help” output with the `-h` option, as in:

```
$ git add -h
```

# Chapter 2. Git Basics

## 2.1 Getting a Git Repository

You can obtain a Git repository in one of two ways:

- take a local directory not under version control and turn it into a Git repository using  the `git init` command.

- clone an existing Git repository from elsewhere. For example,

  ```
  git clone https://github.com/libgit2/libgit2.git
  ```

  If you want to clone the repository into a directory named something other than `libgit2`, you can specify the new directory name as an additional argument:

  ```
  git clone https://github.com/libgit2/libgit2.git mylibgit
  ```

## 2.2 Recording Changes to the Repository

![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)

### 2.2.1 Checking the Status of Your Files

```
git status
```

----

If you want a more simplified output, use `git status -s` or `git status --short` instead.

```
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

The **left-hand column** indicates the status of the **staging area** and the **right-hand column** indicates the status of the **working tree**.

Flags:

- `??`: New files that aren't tracked.
- `A`: New files that have been added to the staging area
- `M`: modified files

In the output example above, the `Rakefile` was modified, staged and then modified again, so there are `M` flags on both columns.

### 2.2.2 Tracking New Files

To begin tracking a new file, you use the command `git add`.

The `git add` command takes a path name for either a file or a directory; if it's a directory, the command adds all the files in that directory recursively.

Furthermore, it may be helpful to think of the meaning of `git add` more as "add precisely this content to the next commit" rather than "add this file to the project".

### 2.2.3 Ignoring Files

Often, there are files that you don't want Git to add or even show you as untracked. In such cases, create a file named `.gitignore` and list the patterns.

The rules for the patterns you can put in the `.gitignore` file are as follows:

- Blank lines or lines starting with `#` are ignored.
- Standard glob patterns work, and will be applied recursively throughout the entire working tree.
- You can start patterns with a forward slash (`/`) to avoid recursivity.
- You can end patterns with a forward slash (`/`) to specify a directory.
- You can negate a pattern by starting it with an exclamation point (`!`).

> Glob patterns are like simplified regular expressions that shells use. 
>
> An asterisk (`*`) matches zero or more characters; `[abc]` matches any character inside the brackets (in this case a, b, or c); a question mark (`?`) matches a single character; and brackets enclosing characters separated by a hyphen (`[0- 9]`) matches any character between them (in this case 0 through 9). You can also use two asterisks to match nested directories; `a/**/z` would match `a/z`, `a/b/z`, `a/b/c/z`, and so on.

Here is an example `.gitignore` file:

```
# ignore all .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in any directory named build
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory and any of its subdirectories
doc/**/*.pdf
```

> GitHub maintains a fairly comprehensive list of good `.gitignore` file examples for dozens of projects and languages at https://github.com/github/gitignore if you want a starting point for your project.

In the simple case, a repository might have a single `.gitignore` file in its root directory, which applies recursively to the entire repository. However, it is also possible to have additional `.gitignore` files in subdirectories. The rules in these nested `.gitignore` files apply only to the files under the directory where they are located.

### 2.2.4 Viewing Your Staged and Unstaged Changes

If you want to know exactly what you changed, not just which files were changed, you can use the `git diff` command.

`git diff` with **no other arguments** compares what is in your working directory with what is in your staging area. The result tells you the changes you've made that you haven't yet staged.

If you want to compare your staged changes to your last commit, use `git diff --staged` or `git diff --cached`.

### 2.2.5 Committing Your Changes

Commit with an inline message:

```
git commit -m "Story 182: fix benchmarks for speed"
```

If you want to skip the staging area, Git provides a simple shortcut.

Adding the `-a` option to the `git commit` command makes Git automatically stage every file that is already tracked before doing the commit, letting you skip the `git add` part.

### 2.2.6 Removing Files

To remove a file from Git, you have to remove it from your tracked files (more accurately, remove it from your staging area) and then commit. 

The `git rm` command does that, and also **removes the file from your working directory** so you don’t see it as an untracked file the next time around.

 If you modified the file or had already added it to the staging area, you must force the removal with the `-f` option. This is a safety feature to prevent accidental removal of data that hasn’t yet been recorded in a snapshot and that can’t be recovered from Git.

Another useful thing you may want to do is to keep the file in your working tree but remove it from your staging area. To do this, use the `--cache` option.

### 2.2.7 Moving Files

After you run the `git mv` command, you need to commit the changes to save them.

```console
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

## 2.3 Viewing the Commit History

We use the command `git log` in this section.

By default, with no arguments, `git log` lists the commits made in that repository in reverse chronological order, that is, the most recent commits show up first.

```
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

One of the more helpful options is `-p` or `--patch`, which shows the difference (the patch output) introduced in each commit. You can also limit the number of log entries displayed, such as `-2` to show only the last two entries.

```
$ git log -p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
 spec = Gem::Specification.new do |s|
     s.platform  =   Gem::Platform::RUBY
     s.name      =   "simplegit"
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
     s.author    =   "Scott Chacon"
     s.email     =   "schacon@gee-mail.com"
     s.summary   =   "A simple gem for using Git in Ruby code."

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
     end

 end
-
-if $0 == __FILE__
-  git = SimpleGit.new
-  puts git.show
-end
```

If you want to see abbreviated stats for each commit, you can use the `--stat` option.

```
$ git log --stat
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

 Rakefile | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

 lib/simplegit.rb | 5 -----
 1 file changed, 5 deletions(-)

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit

 README           |  6 ++++++
 Rakefile         | 23 +++++++++++++++++++++++
 lib/simplegit.rb | 25 +++++++++++++++++++++++++
 3 files changed, 54 insertions(+)
```

Another really useful option is `--pretty`. This option changes the log output to formats other than the default. A few prebuilt option values are available for you to use. For example, the `oneline` value for this option prints each commit on a single line.

```
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
```

Another option value `format` allows you to specify your own log output format.

```
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit
```

For more details, refer to [2.3 Git 基础 - 查看提交历史](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)

----

You can run the `git log` command to show you where the branch pointers are pointing. 

Moreover, by default, `git log` will only show commit history below the branch you've checked out. To show commit history for the desired branch, you have to explicitly specify it: `git log <branch>`. To show all of the branches, add `--all` to your `git log` command.

----

```
$ git log --oneline --decorate --graph --all
* c2b9e (HEAD, master) made other changes
| * 87ab2 (testing) made a change
|/
* f30ab add feature #32 - ability to add new formats to the
* 34ac2 fixed bug #1328 - stack overflow under certain conditions
* 98ca9 initial commit of my project
```

## 2.4 Undoing Things

### 2.4.1 `--amend`

One of the common undos takes place when you commit too early and possibly forget to add some files, or you mess up your commit message. If you want to redo that commit, make the additional changes you forgot, stage them, and commit again using the `--amend`option.

```
git commit --amend
```

After you run the command, the commit-message editor fires up with the message of your previous commit. 

It's important to understand that when you're amending your last commit, you're not so much fixing it as replacing it entirely with a new, improved commit that pushes the old commit out of the way and puts the new commit in its place.

### 2.4.2 Unstaging a Staged File

How can you unstage a staged file? The `git status` command reminds you:

```
$ git add *
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
    modified:   CONTRIBUTING.md
```

So, let's use that adice to unstaged the `CONTRIBUTING.md` file:

```
git reset HEAD CONTRIBUTING.md
```

### 2.4.3 Unmodifying a Modified File

What if you realize that you don't want to keep your changes to the `CONTRIBUTING.md` file? How can you revert it back to what it looked like in the last commit? Still, `git status` tells you how to do it.

```
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

So simply run the following command:

```
git checkout -- CONTRIBUTING.md
```

It's important realize that `git checkout -- <file>` is dangerous command. Git just replaced that file with **last staged or committed version** and the original file cannot be recovered.

---

Actually, anything that is committed in Git can almost always be recovered. Even commits that were on branches that were deleted or commits that were overwritten with an `--amend` commit can be recovered. However, anything you lose that was never committed is likely never to be seen again.

### 2.4.4 Undoing things with `git restore`

Git version 2.23.0 introduced a new command: `git restore`. It's basically an alternative to `git reset`. As of Git version 2.23.0, Git will use `git restore` instead of `git reset` for many undo operations.

Unstaging a stage file with `git restore`:

```
git restore --staged <file>
```

Unmodifying a modified file with `git restore`:

```
git restore <file>
```

You can see these advice when running the `git store` command as of Git 2.23.0.

## 2.5 Working with Remotes

### 2.5.1 Showing Your Remotes

To see which remote servers you have configured, you can run the `git remote` command. If you've cloned your repository, you should at least see `origin`-- the default name Git gives to the server you cloned from.

You can also specify `-v`, which shows you the URLs that Git has stored for the shortname to be used when reading and writing to that remote:

```
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```

### 2.5.2 Adding Remote Repository

```
git remote add <shortname> <url>
```

### 2.5.3 Fetching and Pulling from Your Remotes

To get data from your remote project, run:

```
git fetch <remote>
```

If you clone a repository, the command automatically adds that remote repository under the name "origin".

If your current branch is set up to track a remote branch, you can use the `git pull` command to automatically fetch and then merge that remote branch into your current branch. 

By default, the `git clone` command automatically sets up your local `master`  branch to track the remote `master` branch (or whatever the default branch is called) on the server you cloned from. 

----

It's important to note that `git fetch` command **ONLY** downloads the data to your local repository -- it does **NOT** automatically merge it with any of your work or modify what you're currently working on. You have to merge it manually into your work when you are ready.

### 2.5.4 Pushing to Your Remotes

```
git push <remote> <branch>
```

### 2.5.5 Inspecting a Remote

If you want to see more information about a particular remote, you can use:

```
git remote show <remote>
```

### 2.5.6 Renaming and Removing Remotes

```
git remote rename <old> <new>
git remote remove <name>
```

## 2.6 Tagging

### 2.6.1 Listing Your Tags

If you want the entire list of tag, run the command `git tag`. The use of `-l` or `--list` in this case is optional. The outputs are displayed in alphabetical order.

If you want to search for tags that match a particular pattern, then `-l` or `--list` is mandatory. For example:

```
git tag -l "v1.8.5*"
```

### 2.6.2 Creating Tags

Git supports two types of tags: *lightweight* and *annotated*.

- A lightweight tag is very much like a branch that doesn’t change — it’s just a pointer to a specific commit.
- Annotated tags, however, are stored as full objects in the Git database. 
  They’re checksummed; contain the tagger name, email, and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG). 

----

**Annotated Tags**

Create an annotated tag:

```
git tag -a v.14 -m "my version 1.4"
```

`-a` indicates that it's an annotated tag, and `-m` specifies a tagging message. If you don't specify a message for an annotated tag. Git launches your editor so you can type in it.

You can see the information of tag by using the `git show` command.

```
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

----

**Lightweight Tags**

To create a lightweight tag, just providing a tag is sufficient.

```
$ git tag v1.4-lw
$ git tag
v0.1
v1.3
v1.4
v1.4-lw
v1.5
```

This time, if you run `git show` on the tag, you don't see the extra tag information. The output format of lightweight tags is different from that of annotated tags.

```
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date: Mon Mar 17 21:52:11 2008 -0700
  Change version number
```

### 2.6.3 Tagging Later

To tag a particular commit, you specify the commit checksum (or part of it) at the end of the command. For example:

```
git tag -a v1.2 9fceb02
```

### 2.6.4 Sharing Tags

By default, the `git push` command doesn't transfer tags to remote servers. You have to explicitly push tags to the server.'

To do that, run the following command, just like sharing remote branches:

```
git push <remote> <tagname>
```

If you have a lot tags to push up, use `--tags` options, which will transfer all of your tags to the remote server.

```
git push origin --tags
```

### 2.6.5 Deleting Tags

To delete a tag on your local repository, use

```
git tag -d <tagname>
```

Note this does not remove the tag from any remote servers. There are two common variations for deleting a tag from a remote server:

```
git push <remote> :refs/tags/<tagname>
```

or

```
git push <remote> --delete <tagname>
```

### 2.6.6 Checking out Tags

If you want to view the versions of files a tag is pointing to, you can do a `git checkout` of that tag and this will put your repository in "detached HEAD" state.

## 2.7 Git Aliases

```
git config --global alias.ci commit
```

This means that, for example, instead of typing `git commit`, you just need to type `git ci`.

Maybe you want to run an external command. In this case, start the command with a `!` character. For example,

```
git config --global alias.visual '!gitk'
```

# Chapter 3. Git Branching

A branch in Git is simply a lightweight movable pointer to a commit.

## 3.1 Creating a New Branch

```
git branch <branch-name>
```

This creates a new pointer to same commit you're currently on (`HEAD`).

`HEAD` is a pointer to the local branch you're currently on.

The `git branch` command only created a new branch -- it didn't switch to that newly created branch.

## 3.2 Switching Branches

To switch to an existing branch, run the following command:

```
git checkout <branch>
```

which moves `HEAD` to the `<branch>` branch, and moreover, reverts the files in your working directory back to the snapshot that the branch points to.

To create a new branch and switch to it at the same time, run:

```
git checkout -b <branch-name>
```

If your working directory or staging area has uncommitted changes that conflict with the branch you are checking out, Git won't let you switch branches.

## 3.3 Basic Branching and Merging

### 3.3.1 Fast-forward Merge and Three-way Merge

Let's say we have some branches as follows.

![基于 `master` 分支的紧急问题分支（hotfix branch）。](https://git-scm.com/book/en/v2/images/basic-branching-4.png)

Now we want to merge the `hotfix` branch into the `master` branch. We do it with the `git merge` command.

```
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```

You'll notice the phrase "**fast-forward**" in that merge. It occurs when you try to merge one commit with a commit that can be reached by following the first commit's history. Git simplifies things by moving the pointer forward because there's no divergent work to merge together. 

After merging, the branches should look like this:

![`master` 被快进到 `hotfix`。](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

You may no longer need the branch `hotfix` after the work of merging. You can delete it with the `-d` option to `git branch`.

```
git branch -d hotfix
```

----

Now we decide to merge `iss53` into `master` branch.

```console
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

Note that in this case, Git does a *three-way merge* rather than fast-forwarding, that is, instead of just moving the branch pointer forward, Git creates a new snapshot that results from the two snapshots that branches point to and automatically creates a new commit(called *merge commit*) that points to it.

### 3.3.2 Basic Merge Conflicts

Occasionally, the merging process doesn't go smoothly. If you change the same part of the same file differently in the two branches you're merging, Git won't be able to merge them cleanly.

```
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

To see which files are unmerged at any point after a merge conflict, you can run `git status`:

```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")
```

Git adds standard conflict-resolution markers to the files that have conflicts, so you can open them manually and resolve those conflicts. Your file contains a section that looks something like this.

```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

After you've resolved each of these sections in each conflicted file, run `git add` on each file to mark it as resolved. Staging the file marks it as resolved in Git. You can run `git status` again to verify that all conflicts have been resolved:

```
$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

    modified:   index.html
```

Finally run `git commit` to conclude merge.

## 3.4 Branch Management

### 3.4.1 Viewing Branches

If you run `git branch` with no arguments, you get a simple listing of your current branches.

To see the last commit on each branch, you can run `git branch -v`.

```
$ git branch -v
  iss53   93b412c fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```

To see which branches have merged or not merged into you current branch, run `git branch` with `--merged` or `--no-merged` option. Moreover, you can provide an additional argument to ask about the merge state with respect to some other branch without checking that other branch out first.

To see both remote-tracking and local branches, run `git branch --all` or `git branch -a`.

###　3.4.2 Deleting and Renaming a Branch

To delete a local branch that has been merged, run `git branch -d <branch>`. If not merged, then you need to use `-D` rather than `-d`.

Rename the branch locally with the `git branch --move` command:

```
git branch --move bad-branch-name corrected-branch-name
```

To let others see the corrected branch on the remote, push it:

```
git push --set-upstream origin corrected-branch-name
```

The `git push` command with `--set-upstream` or `-u` means if the push succeeds, then set the uploaded stream as upstream branch.

Now the branch `correct-branch-name` is available on the remote. However, the branch with the bad name is present there, you need to delete it by executing the following command.

```
git push --delete origin bad-branch-name
```

## 3.6 Remote Branches

*Remote-tracking branches* are references to the state of remote branches. They're local references that you can't move. Git moves them for you whenever you do any network communication.

Remote-tracking branch names take the from `<remote>/<branch>`.

### 3.6.1 Fetching

To  synchronize your work with a given remote, you run a `git fetch <remote>` command.

It's important to note that when you do a fetch that brings down new remote-tracking branches, you don't automatically have local, editable copies of them. You need to merge it into your work or create your own branch based on your remote-tracking branch.

### 3.6.2 Pushing

When you need to share a branch with the world, you need to push it up to a remote to which you have write access. To push a branch up, using the following command:

```
git push <remote> <branch>
```

For example, you run `git push origin servefix`. This has the same meaning as `git push origin serverfix:serverfix`, which says "take my serverfix and make it the remote's serverfix".

If you didn't want it to be called `serverfix` on the remote, you could instead run `git push origin serverfix:awesomebranch` to push your local `serverfix` branch to the `awesomebranch` branch on the remote project.

### 3.6.3 Tracking Branches

Checking out a local branch from a remote- tracking branch automatically creates what is called a "*tracking branch*", and the branch it tracks is called an "*upstream branch*".

If you are on a tracking branch and type `git pull`, Git automatically knows which server to fetch from and which branch to merge in.

When you clone a repository, Git automatically creates a `master` branch that tracks `origin/master`. You can set up other tracking branches using either of the following commands:

```
git checkout -b <branch> <remote>/<branch>
git checkout --track origin/serverfix
```

There's a even simpler shortcut if the branch name you're trying to checkout (a) doesn't exist and (b) exactly matches a name on only one remote.

```
git checkout <branch>
```

If you already have a local branch and want to set or change the upstream(must existed on the remote) branch you're tracking, you can use the `-u` or `--set-upstream-to` option.

```
git branch -u origin/serverfix
```

---

If you want to see what tracking branches you have set up, you can use `git branch -vv`.

```
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```

So here we can see that our `iss53` branch is tracking `origin/iss53` and is “ahead” by two, meaning that we have two commits locally that are not pushed to the server. We can also see that our `serverfix` branch is tracking the `server-fix-good` branch on our `teamone` server and is ahead by three and behind by one, meaning that there is one commit on the server we haven’t merged in yet and three commits locally that we haven’t pushed.

**Note that these numbers are only since the last time you fetched from each server.** This command does not reach out to the servers.

### 3.6.4 Pulling

`git pull` is a combination of `git fetch` and `git merge`. 

If you have a tracking branch set up, `git pull` will look up what server and branch your current branch is tracking, fetch from that server and then try to merge in that remote branch.

### 3.6.5 Deleting Remote Branches

For example, delete `serverfix` branch from the server:

```
git push --delete guiorigin serverfix
```

