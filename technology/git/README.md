# Git

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Git is easy to learn and has a tiny footprint with lightning fast performance. It outclasses SCM tools like Subversion, CVS, Perforce, and ClearCase with features like cheap local branching, convenient staging areas, and multiple workflows.

These are our required practices for working with the Git Distributed Version Control System (DCVS).

## 1. Git Reading Material

Don't take our word for it; read some documentation to get better:

- [Pro Git](https://git-scm.com/book/en/v2)
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
- [Git for Computer Scientist](https://eagain.net/articles/git-for-computer-scientists/)
- [The Git Parable](http://tom.preston-werner.com/2009/05/19/the-git-parable.html)

## 2. Continuous Deployment

We operate a continuous deployment (CD) pipeline, powered by GitHub Actions

Implementing CD means that:

- Any commits to the `main` branch will be deployed to **production**, if the build is green.

For this reason, when we are developing new functionality, we do so in **feature branches**, named with a `feature/` prefix. This keeps the work isolated until complete. Please create new feature branches off of the `main` branch, or another feature branch you will merge back into on completion of work. With this in mind, to create a new branch:

```
$ git checkout main
$ git pull
$ git checkout -b feature/my-awesome-new-feature
```

This is also known as the [**GitHub Flow** branching strategy](https://guides.github.com/introduction/flow/).

_TODO: Feature Toggles_

- <https://www.martinfowler.com/articles/feature-toggles.html>
- <https://www.unleash-hosted.com/articles/what-are-the-feature-toggle-best-practices>
- <https://featureflags.io/feature-flags-best-practices/>
- <https://github.com/csswizardry/ama/issues/53>

## 3. Multiple repositories

We have chosen to be a poly-repository development team, rather than maintain a mono-repository. This provides us with more granular control over CI/CD, provides meaningful boundaries between code for services/apps, and allows better control over repository access.

Aim for:

- One conceptual group per repository.

  Does this mean one per product, program, library, class? Only you can say. However, dividing stuff up later is annoying and leads to rewriting public history or duplicative or missing history. Dividing it up correctly beforehand is much better.

- Read access control is at the repo level

  If someone has access to a repository, they have access to the entire repo, all branches, all history, everything. If you need to compartmentalize read access, separate the compartments into different repositories.

- Separate repositories for files that might be needed by multiple projects

  This promotes sharing and code reuse, and is highly recommended.

- Separate repositories for large binary files

  Git doesn’t handle large binary files ideally yet and large repositories can be slow. If you must commit them, separating them out into their own repository can make things more efficient.

- Separate repositories for planned continual history rewrites

  Generally we recommended against rewriting public history. However, there are times when doing that just makes sense. One example might be a cache of pre-built binaries so that most people don’t need to rebuild them. Yet older versions of this cache (or at least older versions not at tag boundaries) may be entirely useless and so you want to pretend they never happened to save space. You can rebase, filter, or squash these unwanted commits away, but this is rewriting history and can cause problem. So if you really must do so, isolate these files into a repository so that at least everything else will not be affected.

## 4. Rules of engagement

When working and committing your work into Git, please abide by the following rules. If in doubt, don't hesitate to ask someone for help or advice. We have these rules to make life easier for the whole team, so we're always willing to help with any issues you might have.

1. Commit early, little, and often.

   In much the same way as any well designed code abides by the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), [Keep It Simple Stupid](https://en.wikipedia.org/wiki/KISS_principle), and [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), so too should your commits. Try to keep them limited to a single concept or change, and get that committed sooner rather than later. Better to have all changes in source control as early as possible, and those changes are easier to deal with if they are atomic. This will make working with merges, cherry-picks, and bi-sects considerably easier in the long-run. This methodology should also be applied when working with Pull Requests (PRs); more on that later.

1. Use the **AngularJS commit message convention**.

   We use the [AngularJS commit message convention](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines) to style our commit messages and make them more useful. To help with this, you can install the [Commitizen CLI tool](https://github.com/commitizen/cz-cli) which will guide you through building good messages on the command line. Alternatively, just conform to the standard in the commit tool of your choice.

1. Don’t change published history

   Once you `git push` your changes to the authoritative upstream repository or otherwise make the commits or tags publicly visible, you should ideally consider those commits immutable. If you later find out that you messed up, make new commits that fix the problems by revert, by patching, etc. but absolutely do NOT rewrite history when that history is shared between users.

1. Always use `git pull --rebase`.

   The default merge strategy behind `git pull` is to `git fetch` followed by a `git merge`. This creates a specific merge commit and ugly merge bubbles in the commit log (check out [`gitk`](https://git-scm.com/docs/gitk) to see them).
   It is much better to use `git pull --rebase` to essentially replay your commits _on top_ of any commits you pull from the remote repository, so that they stay inline in date/time order. This does not create a merge commit, thus keeping the repo cleaner and allowing easier `git bisect` behaviour should you need it.
   You can ensure you always use `git pull --rebase` by running the following command to add this as a preference to your global `.gitconfig`:

   ```
   // Force all new branches in all local repos to automatically use rebase
   $ git config --global branch.autosetuprebase always

   // Force existing branches to use rebase.
   $ git config branch.*branch-name*.rebase true
   ```

1. Use a `git merge` workflow in feature branches

   We tend to favour a `git merge` workflow when merging from `main` or other branches into your current working branch. This is generally to avoid having to overcome the replay of rebased commits on a significant commit chain. In practice, we have found that we end up with less errors in merges when we favour `git merge` over `git rebase`.

   ```
   $ git checkout main
   $ git pull
   $ git checkout feature/some-feature
   $ git merge main
   ```

   Because we have enforced rebase for pull, the above sequence of commands will only create a merge commit as the final step. This will bring the `feature/some-feature` branch up to date with `main`, reducing the number of differences between the two branches and therefore minimising the task to merge the two later on.

## 5. Pull Requests

Open small PRs often. When filling in the PR description, remove any pre-generated lists of commit messages (we can get those elsewhere) and try to answer the following three questions instead:

1. What’s the purpose of the pull request?
1. How was this accomplished?
1. Is there any particular area(s) you want your reviews to focus on/be aware of?

Take some time and effort over it, and litter the diff with comments where you feel further explanation is necessary.

## 6. Periodic maintenance

It's important to perform periodic maintenance on Git repos. Unfortunately, a lot of the problems in a repo are hidden until it's too late. The first two items below should be run on the remote repositories as well as your local repositories.

- Validate your repo is sane (`git fsck`)

  You need not check dangling objects unless you are missing something

- Compact your repo (`git gc` and `git gc --aggressive`)

  This will removed outdated dangling objects (after the two+ week grace period). It will also compress any loose objects git has added since your last gc. git will run a minimal gc automatically after certain commands, but doing a manual gc often (and `--aggressive` every few hundred changesets) will save space and speed git operations.

- Prune your remote tracking branches (`git remote update --prune`)

  This will get rid of any branches that were deleted upstream since you cloned/pruned. It normally isn’t a major problem one way or another, but it might lead to confusion.

- Check your stash for forgotten work (`git stash list`)

  If you don’t do it very often, the context for the stashed work will be forgotten when you finally do stumble on it, creating confusion.
