# Coding and Building

Let's walk through contributing to the Redox subpackage `games`, which is a collection of low-def games. We are going to modify `minesweeper` to display **P** instead of **F** on flagged spots.

## Working with Git

Before starting development, read through [Creating a Proper Pull Request](./ch06-10-creating-proper-pull-requests.html), which describes how the Redox team uses Git.

In this example, we will discuss creating a **fork** of the `games` package, pretending you are going to create a `Merge Request` for your changes. **Don't actually do this**. Only create a fork when you have a permanent change you want to contribute to Redox.

## Using Multiple Windows

For clarity and ease of use, we will be using a couple of `Terminal` windows on your host system, each running a different bash shell instance.
1. The `Build` shell, normally at `~/tryredox/redox` or whatever your base `redox` directory is.
2. The `Coding` shell, normally at `~/tryredox/redox/cookbook/recipes/games/source`.

## Set up your Configuration

To get started, follow the steps in [Including a Program in a Redox Build](./ch05-02-including-programs.html) to include the `games` package in your `myfiles` configuration file. In your `Terminal` window, go to your `redox` base directory and run:

```sh
make qemu
```

On Redox, run `minesweeper` as described in the link above. Type the letter `f` and you will see `F` appear on your screen. Use `Ctrl-Alt-G` to regain control of your cursor, and click the upper right corner to exit QEMU.

Keep the `Terminal` window open. That will be your `Build` shell.

## The Recipe

The `games` package is built in the folder `cookbook/recipes/games`. When you `clone` the `redox` base package, it includes a file `cookbook/recipes/games/recipe.toml`. The recipe tells the toolchain how to get the source and how to build it.

When you build the system and include the `games` package, the toolchain does a `git clone` into the directory `cookbook/recipes/games/source`. Then it builds the package in the directory `cookbook/recipes/games/target`.

Edit the recipe so it does not try to automatically clone the sources. 
- Create a `Terminal` window running `bash` on your host system, which we will call your `Coding` shell.
- Change to the `games` directory.
- Open `recipe.toml` in an editor.
  ```sh
  cd ~/tryredox/redox/cookbook/recipes/games
  gedit recipe.toml &
  ```
- Comment out the `[source]` section at the top of the file.
  ```toml
  # [source]
  # git = "https://gitlab.redox-os.org/redox-os/games.git"
  ```
- Save your changes.

## Git Clone

To set up this package for contributing, do the following in your `Coding` shell.
- Delete the source and target directories in `cookbook/recipes/games`.
- Clone the package into the `source` directory, either specifying it in the `git clone` or by moving it after `clone`.
  ```sh
  rm -rf source target
  git clone https://gitlab.redox-os.org/redox-os/games.git --origin upstream --recursive
  mv games source
  ```
- If you are making a permanent change that you want to contribute, (you are not, **don't actually do this**) at this point you should follow the instructions in [Creating a Proper Pull Request](./ch06-10-creating-proper-pull-requests.html), replacing `redox.git` with `games.git`. Make sure you fork the correct repository, in this case [redox-os/games](https:/gitlab.redox-os.org/redox-os/games).

## Edit your Code

- Using your favorite code editor, make your changes. We use `gedit` in this example, from your `Coding` shell. You can also use [VS Code](#vs-code-tips-and-tricks).
  ```sh
  cd source
  gedit src/minesweeper/main.rs &
  ```
- Search for the line containing the definition of the `FLAGGED` constant (around line 36), and change it to `P`.
  ```
  const FLAGGED: &'static str = "P";
  ```

## Check your Code on Linux

Most Redox applications also work on Linux without being modified. You can build and test your program on Linux.
- From within the `Coding` shell, go to the `source` directory and use `cargo` to check for errors.
  ```sh
  cargo check
  ```
  You could also use `cargo clippy`, but `minesweeper` is not clean enough to pass.
- The `games` package creates more than one executable, so to test `minesweeper` on Linux, you need to specify it to `Cargo`. In the `source` directory, do:
  ```sh
  cargo run --bin minesweeper
  ```

## The Full Rebuild Cycle

After making changes to your package, you should `make rebuild`, which will check for any changes to packages and make a new Redox image. `make all` and `make qemu` do not check for packages that need to be rebuilt, so if you use them, your changes may not be included in the system. Once you are comfortable with this process, you can try [some tricks to save time](#shortening-the-rebuild-cycle).
- Within your `Build` shell, in your `redox` directory, do:

  ```sh
  script build.log
  make rebuild
  exit
  ```

  The [script](https://manpages.ubuntu.com/manpages/jammy/man1/script.1.html) command starts a new shell and logs all the output from the `make` command.
  
  The `exit` command is to exit from `script`. Remember to exit the `script` shell to ensure all log messages are written to `build.log`. There's also a [trick](https://manpages.ubuntu.com/manpages/jammy/man1/script.1.html#signals) to flush the log.

- You can now scan through `build.log` to check for errors. The file is large and contains many ANSI Escape Sequences, so it can be hard to read. However, if you encountered a fatal build error, it will be at the end of the log, so skip to the bottom and scan upwards.

## Test your Changes

In the Redox instance started by `make qemu`, test your changes to `minesweeper`.
- Log in with user `user` and no password.
- Open a `Terminal` window.
- Type `minesweeper`.
- Use your arrow keys or `WSAD` to move to a square and type `f` to set a flag. The character `P` will appear.


Congratulations! You have modified a program and built the system! Next, create a bootable Redox with your change. 
- If you are still running QEMU, type `Ctrl-Alt-G` and click the upper right corner of the Redox window to exit.
- In your `Build` shell, in the `redox` directory, do:
  ```sh
  make live
  ```
In the directory `build/x86_64/myfiles`, you will find the file `livedisk.iso`. Follow the instructions for [Running on Real Hardware](./ch02-03-real-hardware.md) and test out your change.

## Shortening the Rebuild Cycle

To skip some of the steps in a full `rebuild`, here are some tricks.

### Build your Package for Redox

You can build just the `games` package, rather than having `make rebuild` check every package for changes. This can help shorten the build cycle if you are trying to resolve issues such as linking to libraries.
- In your `Build` shell, in the `redox` directory, type:
  ```sh
  make r.games
  ```
  Redox's makefiles have a rule for `r.PACKAGE`, where `PACKAGE` is the name of a Redox package. It will make that package, ready to load into the Redox filesystem.

### Make a New Image

Now that all the packages are built, you can make a Redox image without the step of checking for modifications. 
- In your `Build` shell, in the `redox` directory, do:
  ```sh
  make image
  make qemu
  ```
  `make image` skips building any packages (assuming the last full make succeeded), but it ensures a new image is created, which should include the package you built in the previous step.

### Patch an Image

If you feel the need to skip creating a new image, and you want to directly add a file to the existing Redox image, it is possible to do so. However, this is not recommended. You should use a recipe to make the process repeatable. But here is how to access the Redox image as if it were a Linux filesystem.

- **NOTE:** You must ensure that Redox is not running in QEMU when you do this.

- In your `Build` shell, in the `redox` directory, type:
  ```sh
  make mount
  ```
  The Redox image is now mounted as a directory at `build/x86_64/myfiles/filesystem`.
- Remove the old `minesweeper` and replace it with your new version. In the `Build` shell,
  ```sh
  cd ~/tryredox/redox/build/x86_64/myfiles/filesystem
  rm ./bin/minesweeper
  cp ~/tryredox/redox/cookbook/recipes/games/target/x86_64-unknown-redox/stage/bin/minesweeper ./bin
  ```
- Unmount the filesystem and test your image. **NOTE:** You must unmount before you start QEMU.
  ```sh
  cd ~/tryredox/redox
  make unmount
  make qemu
  ```
  The new version of `minesweeper` is now in your Redox filesystem.

### Checking In your Changes

Don't do this now, but if you were to have permanent changes to contribute to a package, at this point, you would `git push` and create a Merge Request, as described in [Creating a Proper Pull Request](./ch06-10-creating-proper-pull-requests.html).

If you were contributing a new package, such as porting a Rust application to Redox, you would need to check in the `recipe.toml` file. It goes in the `cookbook` subproject. You may also need to modify a filesystem config file, such as `config/demo.toml`. It goes in the `redox` project. You must fork and do a proper Pull Request for each of these projects. Please coordinate with the Redox team via [Chat](./ch06-03-chat.html) before doing this.

## VS Code Tips and Tricks

Although not for every Rustacean, **VS Code** is helpful for those who are working with unfamiliar code. We don't get the benefit of all its features, but the Rust support in VS Code is very good.

If you have not used VS Code with Rust, here's an [overview](https://code.visualstudio.com/docs/languages/rust). VS Code installation instructions are [here](https://code.visualstudio.com/docs/setup/linux).

After installing the `rust-analyzer` extension as described in the overview, you get access to several useful features.
- Inferred types and parameter names as inline hints.
- Peeking at definitions and references.
- Refactoring support.
- Autoformat and clippy on Save (optional).
- Visual Debugger (if your code can run on Linux).
- Compare/Revert against the repository with the Git extension. 

Using VS Code on individual packages works pretty well, although it sometimes take a couple of minutes to kick in. Here are some things to know.

### Start in the "source" dir

In your `Coding` shell, start VS Code, specifying the `source` directory.
  ```sh
  code ~/tryredox/redox/cookbook/recipes/games/source
  ```
Or if you are in the `source` directory, just `code .` with the period meaning the `source` dir.

### Add it to your "Favorites" bar

VS Code remembers the last project you used it on, so typing `code` with no directory or starting it from your Applications window or Favorites bar will go back to that project.

After starting VS Code, right click on the icon and select `Add to Favorites`.

### Wait a Couple of Minutes

You can start working right away, but after a minute or two, you will notice extra text appear in your Rust code with inferred types and parameter names filled in. This additional text is just *hints*, and is not permanently added to your code.

### Save Often

If you have made significant changes, `rust-analyzer` can get confused, but this can usually be fixed by doing `Save All`.

### Don't Use it for the whole of Redox

VS Code cannot grok the gestalt of Redox, so it doesn't work very well if you start it in your `redox` base directory. It can be handy for editing recipes, config and make files. And if you want to see what you have changed in the Redox project, click on the Source Control icon on the far left, then select the file you want to compare against the repository.

### Don't Build the System in a VS Code Terminal

In general, it's not recommended to do a system build from within VS Code. Use your `Build` window. This gives you the flexibility to exit Code without terminating the build.