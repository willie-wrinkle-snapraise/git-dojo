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
  * Once you've made your choices, you'll be taken through the rebasing process. It's less fraught than it might seem, and the prompts are helpful. The key things to remember are to make frequent and aggressive use of `git status`, use `git rebase --commit` instead of `git commit`, and to `git rebase --abort` as soon as you suspect you might need to (#failfast.)
  * Get the `<commit>` id from the history--either by `git log` or by looking on GitHub. If there's a specific commit you're trying to address, make sure to start the rebase one commit _before_ that commit.
  * Before rebasing (or doing any other Spooky Advanced Git Nonsense), _cut a new branch_. It's easy, and if you make a mistake you can switch back to your old branch without harm.

### Cherry-picking
* `git cherry-pick <commit>` will apply `<commit>` to your current branch. In a perfect world, this is nearly useless. In the actual world, this is a neat trick you can use to pull an individual commit out of another branch without merging the entire thing. You can get the `<commit>` id from the history (like with rebasing) and once you cherry-pick, you'll be taken through a process very similar to the rebase process--`git cherry-pick --commit` to commit, `git cherry-pick --abort` to abort. Like rebasing, or other advanced "backdoor" git tricks, don't be afraid to cut a new branch to work in if you have any concerns.


### Fast-forwarding and force-pushing
After you rebase or cherry-pick or do other Spooky Advanced Git Nonsense that directly messes with the history, you'll likely get an error message about "fast-forwarding" when you try to [`push`](#pulling-and-pushing) your changes. This is due to how git fundamentally works--the history is _part of the repo_, shared by everyone who's cloned the repo just as much as the files it contains. As such, when you change the history of a branch locally, it conflicts with the history of the branch on the remote. The solution is to "force push" your changes with the `-f` flag.

**WARNING**: this is one of the few ways you can irrevocably delete/mess up things in a git repo. **BE CAREFUL** AND _NEVER_ FORCE-PUSH TO `main`!

You may also see the same "fast-forwarding" error if you've made changes to a branch that someone else has also made changes to. In that case, the solution is to pull the other changes from the remote into your local copy and resolve any conflicts that result. Don't force-push to a branch someone else is working on, you'll erase their changes like a jerk!

## OH NO A MISTAKE
THIS IS A MISTAKE AND SHOULD BE DELETED :(((((((


## Configuring Git
* `git config` lets you configure git (the program).
  * `git config --global` is for global (system-wide) settings. `git config` (without `--global`) configures settings for the repo you're currently in.
  * The general pattern is `git config [--global] <section>.<setting> <value>`, e.g. `git config --global user.name "Sam Raker"`. The ones you want to make sure are configured are
    * `user.name`
    * `user.email`
    * `core.editor` determines which editor is used when editing commit messages and the history when you `rebase -i`. I don't know what the default is for Mac but it's probably `nano` or something. Change it to `vim` or `emacs` if you want; changing it to a whole IDE like vscode is probably not worth it.
* `git config --list` will show you all your settings. There are a lot of them; I don't know what most of them do. Like a lot of things git, they don't really matter until they do, and then you can learn something.

### Other Configuration
* Paths listed in a file called `.gitignore` in the top-level directory of a repository will be ignored by git. (Remember to `add` and `commit` it!)
  * `.gitignore` files have their own peculiar syntax, similar to [globs](https://en.wikipedia.org/wiki/Glob_(programming)).
    * `foo` will match (and therefore ignore) a file or directory named `foo` in the top-level directory
    * `foo/` will match a directory named `foo` in the top-level directory
    * `foo/**/bar` will match a file or directory named `bar` within _any number of other directories_ beneath `foo` (`foo/bar`, `foo/a/bar`, `foo/b/bar`, `foo/a/b/c/d/bar`, etc.)
    * `**/foo` will match a file or directory named `foo` _anywhere_ within the repo, regardless of directory structure.
    * `?` matches any single (non-`/`) character, so `foo/b?r` would match `foo/bar`, `foo/bbr`, etc. (but _not_ `foo/b/r`.)
    * `!` "un-ignores" files and directories, so `!foo/bar` means `bar` will _not_ be ignored even if everything else under `foo` is.
  * Many tools (`cargo new`, `create-react-app`, etc.) generate their own `.gitignore` files, which you can use as inspiration.
* `.gitignore` files exist within and for the repo. Everyone who clones the repo will have the same `.gitignore` file. As such, it's considered good practice not to fill them with platform- or user-specific things (e.g. `.DS_STORE`). Instead, you can add things like that to the file `.git/info/exclude`, which is specific to your own copy of the repo--it's not committed or shared.



## Misc
* `git status` will show you the paths that have been created and staged-but-not-committed. I use it compulsively.
* `git diff <path>` will show you the changes made to an _unstaged_ file.
  * `git diff HEAD <path>` (all caps, important) will show you the changes made to a _staged_ file.
* `HEAD` is a "symbolic ref" that means, basically, "the last commit."
  * `HEAD~<N>` refers to `N` commits _before_ the last commit. So if you want to compare the current state of a file to its state 2 commits ago, you would use `git diff HEAD~2`.
    * There's also `HEAD^<N>` and `HEAD^^` and some other ways of referring to previous commits that take merges into account. [This](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git) is a pretty good discussion on SO if you're curious.


## Afterword
Git is huge and scary and has absolutely _terrible_ UX. Everybody knows this but it's somehow already too late to do anything about it.

Like databases and filesystems, there are a lot of GUI tools out there that purport to make git, or parts of it, easier. Use them if you want, but it's worth developing at least a passing comfort with command-line git in case you're ever in a situation where your GUI of choice is unavailable.

Google aggressively, copy/paste from Stack Overflow, learn _just enough_ to be able to do your job and **do not feel bad about it**. I'd put git in the same bucket as regular expressions, one tier up from `make` or `sed`: you absolutely need to know _about_ it; you almost certainly should be familiar with it; you _can_ devote yourself to becoming an expert in it, if you really want, but the "worth it" curve levels off pretty sharply.

Unfortunately, git is a real Linux Bro tool written and maintained by real Linux Bro dudes and therefore the `man` pages and official documentation are nearly worthless. You're much better off using [bro](http://bropages.org/) or other sources of concrete examples.


Resist the urge to `git add .`

It's nearly impossible to accidentally permanently screw things up. Don't be shy about cutting new branches, and push frequently even if you're not "done".

If you want a (much) deeper dive, [Building Git](https://shop.jcoglan.com/building-git/) by James Coglan is a fun tour through git internals, by systematically (re)building it in Ruby.
