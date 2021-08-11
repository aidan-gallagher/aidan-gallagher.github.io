---
title: "Clean Git History"
excerpt: "Rewriting history to maintain a clean log"
categories:
  - Blog
tags:
  - Git
---

## Introduction

Writing good commit messages are important for

- Reviewing pull requests
- Maintaining code (understanding why a specific feature was added)
- Fixing bugs
- Generating release notes

## The Problem

You will not always write great commit messages first time. This can be due to 2 reasons.

Firstly, you may decide to adapt your solution as you attempt to implement it. This pivot may invalidate previous code changes and therefore their commit messages.

Secondly, when you are programming in the zone you may not want to break your flow to document your change. This can result in short and untidy commit messages.

## Rewrite History

Fortunately git supports rewriting history. This means your first attempt at writing the commit message does not have to be your final version.

[Warning - only do this on a branch that you are working on alone]

With this knowledge your first draft at commits can be quick and provide just enough information that it is useful to just youself.

Once you are confident of your implementation it is time to clean up the commit history. With the interactive rebase tool, you can then stop after each commit you want to modify and change the message, add files, or do whatever you wish.

You need to tell the rebase command how far back you would like to go. An eay way to do this is specify how many commits back from HEAD. This example will go back 10 commits

```
git rebase -i HEAD~10
```

A better solution is to use `merge-base` to find the commit where the feature branch branched off from master.

```
git rebase -i `git merge-base HEAD master`
```

If you are not familiar with the interactive rebase command then please read the chapter [Git Tools - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) from the Pro Git book.

### Commit hunks, not just files.

Your commits should be split up into logical changes. Just because you make 2 changes in the same file does not mean they have to be in the same commit. This can be achieved by using the `--patch` option (`git add --patch`)

Or if you prefer a less archaic method, it can be done in VSCode.

1. Open up the Source Control view, then click a changed file to open the comparison.
2. Select the lines you want to stage.
3. Right click -> Stage Selected Ranges

## Reviewing a PR

Your PR will now be split up into multiple neat commits. For example commit 1 will be a refactor that has no functional change, commit 2 introduces a new feature and commit 3 fixes typos and other small things you found along the way.

You should encourage your reviewers to step through each commit when reviewing the code.

## Re-reviewing a PR

After making changes based off pull request feedback you do not want throw away the clean commit history with commit messages like "fixing various PR comments".
You can continue to use interactive rebase however when using this approach the reviewers revisiting the PR can not easily see what changes you have implemented since the previous review.
Rather than seeing a single commit like this.
![normal_commit_update](/assets/2021-03-01-clean-git-history/normal_commit_update.png)  
(Note: Bitbucket strips out newlines in the preview so the title and body are merged into 1 long line.)

They will see a series of commits have been removed and then readded.
![rebase_commit_update](/assets/2021-03-01-clean-git-history/rebase_commit_update.png)

Bitbucket does not provide a way of viewing the change. What you are trying to view is a comparison of 2 commits; the commits being HEAD before the rewrite and HEAD after the rewrite.

Find the 2 commit's SHA in bitbucket.
![rebase_commit_update_annotated](/assets/2021-03-01-clean-git-history/rebase_commit_update_annotated.png)
Then use git to compare them  
```
git diff before_SHA after_SHA
```
![git_diff.png](/assets/2021-03-01-clean-git-history/git_diff.png)

This is a bit of an archaic view and not good for multiple files.
Modern tools such as VSCode provide a much better experience.
![vscdoe-comparison](/assets/2021-03-01-clean-git-history/vscode-comparison.png)

To set it up

1. Install [folder compare extension](https://marketplace.visualstudio.com/items?itemName=moshfeu.compare-folders) for VSCode.

```
code --install-extension moshfeu.compare-folders
```

2. Configure vscode as the git difftool. Add the following to ~/.gitconfig

```
[diff]
    tool = default-difftool
[difftool "default-difftool"]
    cmd = code --wait --new-window --diff $LOCAL $REMOTE
```

3. Git diff the two commits  
   (You may wish to create a bash alias for this.)

```
COMPARE_FOLDERS=DIFF git difftool --dir-diff commit1_SHA commit2_SHA
```

### Orphaned commits on remote server
The previous step assumes you have access to both commits - the removed commit and the added commit. You will always have access to the added commit since it is referenced to by the branch. The removed commit is now a orphan which means there are no references to it. 

## Enforcing Good Commits

A tool can be used to ensure all commit messages are uniform.

1. Install [Gitlint](https://github.com/jorisroovers/gitlint) on your jenkins server

```
pip install gitlint
```

2. Add a Gitlint stage to your pipeline

```
stage('Gitlint') {
    steps {
        dir("${SRC_DIR}") {
            sh "gitlint --commits upstream/${env.CHANGE_TARGET}..origin/${env.BRANCH_NAME}"
        }
    }
}
```
