---
title: "Clean Git History"
excerpt: "Rewriting history to maintain a clean log"
categories:
  - Blog
tags:
  - Git
  - Infrastructure
---

## Introduction

Writing good commit messages are important for

- Reviewing pull requests
- Maintaining code
- Fixing bugs
- Generating release notes

## The Problem

You will not always write great commit messages first time. This can be due to 2 reasons.

Firstly, you may decide to adapt your solution as you attempt to implement it. This pivot may invalidate previous code changes and therefore their commit messages.

Secondly, when you are programming in the zone you may not want to break your flow to document your change. This can result in short and untidy commit messages.

## Rewrite History

Fortunately git supports rewriting history. This means your first attempt at writing the commit message does not have to be your final version.

<p class="notice --warning">
  Warning: only rewrite git history on a branch that you are working on alone
</p>

With this knowledge your first draft at commits can be quick and provide just enough information to be useful to yourself.

Once you are confident of your implementation it is time to clean up the commit history. With the interactive rebase tool, you can then stop after each commit you want to modify and change the message, add files, or do whatever you wish.

You need to tell the rebase command how far back you would like to go. An eay way to do this is specify how many commits back from the most recent to start at. This example will go back 10 commits

```
git rebase -i HEAD~10
```

Another method is to use `merge-base` to find the commit where the feature branch branched off from master and start from there.

```
git rebase -i `git merge-base HEAD master`
```

If you are not familiar with the interactive rebase command then please read the chapter [Git Tools - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) from the Pro Git book.

### Commit hunks, not just files.

Your commits should be split up into logical changes. Just because you make 2 changes in the same file does not mean they have to be in the same commit. This can be achieved by using the `--patch` option (`git add --patch`)

To do this in VSCode:

1. Open up the Source Control view, then click a changed file to open the comparison.
2. Select/highlight the lines you want to stage.
3. Right click -> Stage Selected Ranges.

## Reviewing a PR

Your PR will now be split up into multiple neat commits. An example of logical commit changes may be

- Commit 1: Refactor that has no functional change but is necessary to support the feature to be developed.
- Commit 2: Introduce the new feature.
- Commit 3: General clean up & typo fixes carried out in unrelated areas of the repository.

The reviewers should be encouraged to step through each commit individually and look at each commit's message.

## Re-reviewing a PR

After making changes based off pull request feedback you do not want throw away the clean commit history with commit messages like "fixing various PR comments".

You can continue to use interactive rebase tool, however, when using this approach the reviewers revisiting the PR can not easily see what changes you have implemented since the previous review.
Rather than seeing a single commit like this.
![normal_commit_update](/assets/2021-03-01-clean-git-history/normal_commit_update.png)  
(Note: Bitbucket strips out newlines in the preview so the title and body are merged into 1 long line.)

They will see a series of commits have been removed and new ones that have been added. This is because git will generated a new hash (unique ID) for every commit after the changed commit.
![rebase_commit_update](/assets/2021-03-01-clean-git-history/rebase_commit_update.png)

Bitbucket does not provide a way of viewing the change between the original commit on the feature branch and the new commit on the feature branch. Instead bitbucket will only show the change between the feature branch and the target branch (usually master).

Locally you can view the comparison of 2 commits using `git diff` (or VSCode). On bitbucket you can find the SHA of HEAD before the rewrite (the most recently removed commit) and HEAD after the rewrite (the most recently added commit). You will always have access to the added commit since it is referred to by the branch. However, the removed commit is now a orphan, which means even though it exists on the server, `git pull` will not pull it down since the commit has no references. This means unless your development team agrees on some work around you can not guarantee reviewers will be able to compare your updates.

The approach I take is to reply to each PR comment that requires action with the commit hash that I have addressed the issue in.

## Enforcing Good Commits

To maintain quality git commit messages the gitlint tool can be used.

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

## Summary

- Good commit messages are important.
- Commits can be cleaned up after the fact.
- Rewriting commits after raising a PR can make reviewing harder.
- Gitlint can be used in CI pipeline to automatically check commit quality.
