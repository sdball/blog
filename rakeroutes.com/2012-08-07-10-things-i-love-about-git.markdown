---
title: "10 Things I Love About Git"
date: 2012-08-07 12:00
---

Not everyone loves git. It's true! But I do, and here are some reasons why.

## 1. Branching for experiments

I want to experiment with rewriting some code and make commits while I'm doing it.

In git I'd be able to just make a local branch and hack away. If I'm successful then I can just push up the branch to the team's repository for code review. And that's just one use case! Branches are just, so easy in git that it opens up so many possibilities for workflow that the community has come up with lots of different ways (e.g. git-flow, github-flow) for a team to code.

In svn I have to do without commits as I'm developing, make a branch on the remote repo (because there is only a remote repo), or copy out the files to some other directory on my computer (yuck!) for this experimental refactor.

## 2. Partial commits

Oh no! I've just written too much code without committing.

In git I have a staging area to serve as an intermediate place for code that I'm ready to commit up to the public repository. Edited file X and made a fix to file Y? No problem. Commit file X, then commit file Y.

In svn there is no concept of partial commits. I have to fully commit my current changes to the repo (because there is only a remote repo).

**EDIT**: Ha, no. Subversion doesn't have partial file commits, but you totally can choose individual files to commit. (derp)

**EDIT 2**: Aw man. I thought I had a bit here about `git add -i` and `git add -p`. Well I don't and it's even more awesome than simply commiting individual files. `git add -i` enters into a console GUI that allows you to do things like add untracked files and enter patch mode on individual files. In patch mode git will go through the changed sections of a single file and ask if they should be included in the commit. If you've done a dozen edits to a single file but want to commit just a couple of them (atomic commits are nice) then patch mode is what you want. `git add -p` goes into patch mode directly.

**AWESOME**: Molecule on Hacker News [explained `git add –edit`](https://news.ycombinator.com/item?id=4382779), which I had never heard of. `git add --edit` opens an individual file or all of your changes in a single diff (like you would see in git diff) that you can edit. Changed lines you delete from this diff won't be staged (but will still be waiting to be committed). This is really handy if you want to break up a cluster of changed lines that git is considering a single edit in patch mode.

## 3. Everyone has the repo

Git not only allows me to easily experiment with the code; it allows me to experiment with the repo itself. "What will happen when I merge branch X to branch Y?"

In git I can try everything locally and back out if things get dicey.

In svn you cross your fingers: you'll either have no trouble or be immersed into a world of conflict resolution.

## 4. Resolving conflicts is way easier (than svn)

In git if I have a private branch from a branch that has been updated with new (conflicting) commits I can rebase its commits one at at time against the public destination branch. I can resolve conflicts as they arise between my code and the current codebase. This makes dealing with conflicts easy because I get the context of the conflict (my commit message) and only see one conflict at a time.

In svn if I merge a branch against another and there are a lot of conflicts there's nothing I can do but resolve them all at the same time. What a mess.

## 5. I can put my work aside

I'm in a feature branch hacking away on a spike and have a dozen modified files (remember, spike) and let's set this up equally and say I haven't made any commits yet. Uh oh, we have a critical bug in production that I need to fix NOW.

In git? No problem! I git stash away all my changes (including files that I haven't committed yet), checkout the master branch, create a local branch for the bug and get to work. When the work is done and master is deployed. I switch back to my spike branch and restore my work exactly as I left it and pull in the bugfix code to be sure my spike is accurate.

In svn? I have to make a new checkout of the repository to get a clean state. Fix the code there and commit up. Then I either get to rewrite the bugfix code into the checkout that has the spike code, or rewrite the spike code into the current checkout that has the bugfix code.

## 6. Actual branches and tags

Git has actual semantic branches and tags. The repository can can authoritatively say that this is a branch and that is a tag and treat them differently.

Svn has copies – copies! – of the repository directory in a folder called "branches" or a folder called "tags". There is no difference between them and no requirement to even have the folders. You don't even need to have the folder "trunk", that's just a popular convention.

## 7. Merge metadata

Git optionally (if you have a merge commit or not) records a ton of metadata about a merge (author, betweenranch, conflicting files, commits brought in by the merge, etc). git also automatically handles not merging commits that are already in the destination branch.

Svn records nothing and you get to play the fun game of writing down the starting commit number and putting it into your merge command e.g. (`-r 11235:HEAD`) it you want to avoid pointless conflicts.

Edit: It looks like in 1.5+ [svn has branch merge metadata](https://news.ycombinator.com/item?id=4381285), but it's [apparently buggy](https://news.ycombinator.com/item?id=4381303).

## 8. Github

Yes, github isn't actually part of git but it may as well be. I believe that git became the default choice for version control because github is so excellent.

Subversion has no github because subversion can't have a github. It simply lacks the capabilities to allow hundreds of users to sporadically collaborate to a project in any meaningful way.

No github means no pull requests. No pull requests mean no easy code review. No pull requests mean you either give users write access to the repo or you don't. No pull requests mean you don't get conversations about code right in the repository.

EDIT: Nope I'm wrong…kind of. Subversion does have a github: github itself! Github provides [SVN access to any repo](https://github.com/blog/1178-collaborating-on-github-with-subversion) hosted on github. Of course it's still using git internally and providing access to subversion clients. Subversion does lack the capabilities to allow hundreds of users to collaborate, but github allows it to use git without having to use git. Impressive. Jump down to [Igor Mosyagin's quick tutorial](http://rakeroutes.com/blog/10-things-i-love-about-git/#comment-619138938) in the comments to get a run through. Thanks [Igor](https://twitter.com/shrimpsizemoose)!

## 9. Bisect for bug hunting awesomeness

Git bisect lets you crawl through your repos history to find out when some bug (although it could be any change) was introduced. I'm not going to go into the details but the steps are essential: start a bisect, mark a version of the repo as "good" and mark a version of the repo as "bad". Then you can either manually tell git whether or not the versions are good or bad as it steps through the different commits, or give it a shell command to be able to run for itself. At the end you'll have the exact commit where the bug was introduced.

EDIT: Subversion doesn't have bisect, but trust in CPAN for it provides an [svn-bisect perl library](http://search.cpan.org/~infinoid/App-SVN-Bisect-1.1/bin/svn-bisect). Very nice.

## 10. The reflog saves you from yourself

In git there is a repo log which holds the history of all the commits you have made, and there is also a reflog that holds almost all the **actions** you have done locally in your repo. All of them. Did you just change branches? You've got an entry in your reflog. Did you just make a commit? Reflog. Did you just completely blow away a day's work with a git reset –hard? Reflog.

What good is it to have actions logged in the reflog? Why, you can use it to undo any action you've done locally to your repo. Seriously. Even a git reset –hard. Even a completely crazy merge.

Even if you delete a branch with -D you can still recover your work by rescuing the commits from your reflog. Although in that case you'll probably have a better time using git fsck to find the commits that have been left dangling.

**EDIT**: Almost any action. You can use it to undo almost any action in your repo. [Dubiousjim rightly points out](http://rakeroutes.com/blog/10-things-i-love-about-git/#comment-619145311) that there's no way to recover uncommitted changes from a `git reset --hard`. Yep, absolutely no way to recover those commits. (Now awaiting a git/filesystem wizard to share an amazing technique involving low level disk access to prove that wrong.) :-)

