# Git Dojo

## About Git
Git is a [`version control system`](https://en.wikipedia.org/wiki/Version_control), initially created by [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) in 2005 to facilitate development of the Linux kernel. Since then, it's become nearly omnipresent.

## Initialize a new repo
* `git init`
  * if you have `>= 2.28` (`git --version` to check) you can change the default branch name
    * pass `-b main`/`--initial-branch=main` to initialize the new repo with `main` as the default branch
    * use [`git config`](#configuring-git) `--global init.defaultBranch main`

There's nothing special about `main`, it's just what most people use. (Some people use `trunk`.)

## Remotes
Remotes are places where your code lives. Most of the time, this will be on github, but it _doesn't have to be_. Other options include [gitlab](https://about.gitlab.com/); you can even host your own git server.
* `git remote -v` will show you a list of remotes
* `git remote add <name> <url>` will add a remote
  * `name` is how you identify the remote to git, when you e.g. [`pull`](#pulling-and-pushing) or [`push`](#pulling-and-pushing)
* `git remote set-url <name> <url>` will update the url of a remote by name
* `git remote remove <name>` will remove a repo by name

Much like most people use `main` for the default branch, it's conventional to call your primary/default remote `origin`.


## Pulling and Pushing
Much like branches, individual git repositories (remotes) are discrete and do not, by default, affect one another. In order to interact with other instances of your repo, you need to `push` and `pull`.
* `git pull <remote> <branch>` will "pull" the current state of a given branch on a given remote and [`merge`](#merging) it into your current branch. Thus, if you wanted to pull the latest changes that have been made to `main` on `origin` into the branch you're working on, you could do `git pull origin main`.
* `git push <remote> [<branch>]` will "push" the changes committed in a given branch to the given remote. If you leave out `branch` it will push the branch you're currently on. Thus, once you've committed your local changes, you could do `git push origin` to push those changes up to `origin`. (NB: separate branches are still separate, even on `origin`. Someone would still have to [`merge`](#merging) your branch into e.g. `main`.)

### Upstreams and tracking
By default, everything is disconnected and you have to specify remotes and branches manually. You can, however, connect local branches to branches on remote servers.
* `git branch --set-upstream-to=<remote>/<upstream-branch> <local-branch>` will set `<upstream-branch>` on `<remote>` as the `upstream` branch for `<local-branch>`. After doing this, you can omit `<remote>` and `<branch>` when you `pull` or `push` the branch whose upstream you configured.
* `git push -u <remote> <upstream-branch>` will push your current branch to `<remote>` _and also_ create `<upstream-branch>` on `<remote>` and configure it as the upstream for your current branch; it's the equivalent of doing `git push <remote>` immediately followed by `git branch --set-upstream-to=<remote>/<local-branch> <local-branch>`.

### Fetching
`git pull`, as discussed, does two things in sequence--it gets the latest changes to one branch from the remote _and_ applies them to your current branch. You can, however, _just_ retrieve changes from a remote, _without_ applying them to a local branch.
* `git fetch <remote>` gets changes from _all_ branches on `<remote>`. It's useful for seeing what other work has been done on the remote and potentially interacting with those branches locally, without making any changes on the remote (remember, anything you do locally is separate until you `push`.)


### Some other stuff
* `git push <remote> :<branch>` will delete `<branch>` on `<remote>`. You probably don't want to do this.
  * `git branch -D <branch>` will delete `<branch>` locally. This is marginally more useful.
* `git push <remote> <local-branch>:<remote-branch>` will push `<local-branch>` to `<remote>` _as_ `<remote-branch>` (e.g. when you `fetch` it'll show up as `<remote-branch>`.) The only situation I've found this to be relevant is with Heroku, which will only trigger a rebuild/redeploy for branches named `main`, so you have to do e.g. `git push heroku my-branch:main`.
* `git checkout <remote>/<branch>` will change you immediately to be on the version of `<branch>` that's on `<remote>` (provided you `fetch <remote>` first.) Be warned--as the notice you'll see will tell you, this will put you in `detached head state` so you should `git switch -c` immediately. `git checkout a-remote/a-branch` is basically the equivalent of    
```bash
$ git switch -c a-branch
$ git branch --set-upstream-to a-remote/a-branch
$ git pull a-remote a-branch
```    
It can be useful from time to time, especially if you need to quickly make a local copy of a remote branch (in which case I recommend making the name of the new branch you create something distinct.)

## History
Every commit is recorded in the history git keeps, also known as the `log`.
* `git log` will show you the history. `git log` shows a lot of information, and has _a lot_ of flags/arguments that can be passed to it. Definitely check out `git log --help`, but be aware that help gets into some pretty confusing/advanced git concepts.
  * `git log --grep <pat>` will filter the history, showing only commits that match the regular expression `<pat>`. You can also pass `-i` to make `<pat>` match case-insensitively.
  * `git log -p` shows a patch (i.e., what's been changed) for each commit in the history.
  * `git log --oneline` shows each commit message flattened into one line

You can combine flags (where it makes sense), e.g. `git log --grep 'Add' --oneline`

### Rebasing
The history is (ostensibly, deontologically) immutable. You can, however, edit it.
* `git rebase -i <commit>` will let you edit the history. You _almost never_ want to do this, and could really mess things up if you do it wrong.
  * Once you've opened the editor, you can
    * Re-order commits (by moving them around)
    * Delete commits (by changing `pick` to `d`
    * Squash commits together (by changing `pick` to `s`)
    * Edit commits (by changing `pick` to `e`)
  * Once you've made your choices, you'll be taken through the rebasing process. It's less fraught than it might seem, and the prompts are helpful. The key things to remember are to make frequent and aggressive use of `git status`, and to `git rebase --abort` as soon as you suspect you might need to (#failfast.)
  * Get the `<commit>` id from the history--either by `git log` or by looking on GitHub. If there's a specific commit you're trying to address, make sure to start the rebase one commit _before_ that commit.
  * Before rebasing (or doing any other Spooky Advanced Git Nonsense), _cut a new branch_. It's easy, and if you make a mistake you can switch back to your old branch without harm.

## OH NO A MISTAKE
THIS IS A MISTAKE AND SHOULD BE DELETED :(((((((


## Misc
* `git diff <path>` will show you the changes made to an _unstaged_ file.
  * `git diff HEAD <path>` (all caps, important) will show you the changes made to a _staged_ file.
* `HEAD` is a "symbolic ref" that means, basically, "the last commit."
  * `HEAD~<N>` refers to `N` commits _before_ the last commit. So if you want to compare the current state of a file to its state 2 commits ago, you would use `git diff HEAD~2`.
    * There's also `HEAD^<N>` and `HEAD^^` and some other ways of referring to previous commits that take into account merges. [This](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git) is a pretty good discussion on SO if you're curious.


## Afterword
Git is huge and scary and has absolutely _terrible_ UX. Everybody knows this but it's somehow already too late to do anything about it.

Google aggressively, copy/paste from Stack Overflow, learn _just enough_ to be able to do your job and _do not feel bad about it_. I'd put it in the same bucket as regular expressions, one tier up from `make` or `sed`. 

The good news is that it's nearly impossible to accidentally permanently screw things up. Don't be shy about cutting new branches, and push frequently even if you're not "done".
