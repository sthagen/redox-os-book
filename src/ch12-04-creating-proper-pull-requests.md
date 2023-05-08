# Creating Proper Pull Requests

It's completely fine to just submit a small pull request without first making an issue, but if it's a big change that will require a lot of planning and reviewing, it's best you start with writing an issue first.

The steps given below are for the main Redox project - submodules and other projects may vary, though most of the approach is the same.

## Using Git in terminal

1. In an appropriate directory, e.g. `~/tryredox`, clone the Redox repository to your computer using one of the following commands:
  - HTTPS: 
    ```sh
    git clone https://gitlab.redox-os.org/redox-os/redox.git --origin upstream --recursive
    ```
  - SSH: 
    ```sh
    git clone git@gitlab.redox-os.org:redox-os/redox.git --origin upstream --recursive
    ```
  - Use HTTPS if you don't know which one to use. (Recommended: learn about [SSH](./ch12-01-signing-in-to-gitlab.md#using-ssh-for-your-repo) if you don't want to have to login every time you push/pull!)
  - If you used `bootstrap.sh` (see [Building Redox](./ch02-05-building-redox.md)), the `git clone` was done for you and you can skip this step.
2. Change to the newly created redox directory and rebase to ensure you're using the latest changes: 
    ```sh
    cd redox
    git rebase upstream master
    ```
3. You should have a fork of the repository on GitLab and a local copy on your computer. The local copy should have two remotes; `upstream` and `origin`, `upstream` should be set to the main repository and `origin` should be your fork. Log into Redox Gitlab and fork the [Repository](https://gitlab.redox-os.org/redox-os/redox) - look for the button in the upper right.
4. Add your fork to your list of git remotes with
  - HTTPS: 
    ```sh
    git remote add origin https://gitlab.redox-os.org/MY_USERNAME/redox.git
    ```
  - SSH: 
    ```sh
    git remote add origin git@gitlab.redox-os.org:MY_USERNAME/redox.git
    ```
  - Note: If you made an error in your `git remote` command, use `git remote remove origin` and try again.
5. Alternatively, if you already have a fork and copy of the repo, you can simply check to make sure you're up-to-date. Fetch the upstream, rebase with local commits, and update the submodules:
    ```sh
    git fetch upstream master
    git rebase upstream/master
    git submodule update --recursive --init
    ```
    Usually, when syncing your local copy with the master branch, you will want to rebase instead of merge. This is because it will create duplicate commits that don't actually do anything when merged into the master branch.
6. Before you start to make changes, you will want to create a separate branch, and keep the `master` branch of your fork identical to the main repository, so that you can compare your changes with the main branch and test out a more stable build if you need to. Create a separate branch:
    ```sh
    git checkout -b MY_BRANCH
    ```
7. Make your changes and test them.
8. Commit:
    ```sh
    git add . --all
    git commit -m "COMMIT MESSAGE"
    ```
    Commit messages should describe their changes in present-tense, e.g. "`Add stuff to file.ext`" instead of "`added stuff to file.ext`".
    Try to remove duplicate/merge commits from PRs as these clutter up history, and may make it hard to read.
9.  Optionally run [rustfmt](https://github.com/rust-lang/rustfmt) on the files you changed and commit again if it did anything (check with `git diff` first).
10. Test your changes with `make qemu` or `make virtualbox`.
11. Pull from upstream:
    ```sh
    git fetch upstream
    git rebase upstream/master
    ```
  - Note: try not to use `git pull`, it is equivalent to doing `git fetch upstream; git merge master upstream/master`.
12. Repeat step 10 to make sure the rebase still builds and starts.
13. Push your changes to your fork:
    ```sh
    git push origin MY_BRANCH
    ```
14. On [Redox GitLab](https://gitlab.redox-os.org/), create a Merge Request, following the template. Describe your changes. **Submit!**

15. If your merge requests is ready, send the link on [Redox Merge Requests](https://matrix.to/#/#redox-mrs:matrix.org) room.

## Using GitLab web interface

1. Open the project repository that you want and click in "Fork" to create your fork.
1. Click in "Web IDE" to open the GitLab IDE for browsers and make your changes.
1. Click in "Create commit" to apply your changes (try to not flood your fork with little commits, it's Ok to make bigger commits).
1. Return to the main page of your fork (you can click on the name in the top left position inside of Web IDE).
1. Click in the "Merge Requests" on the left side.
1. Select your fork branch and create the merge request.
1. If your merge requests is ready, send the link on [Redox Merge Requests](https://matrix.to/#/#redox-mrs:matrix.org) room.

- Remember that you will need to update your forks manually in web interface (delete the fork/create a new fork if the upstream repository push commits from other contributors).