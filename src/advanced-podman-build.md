# Advanced Podman Build

To make the Redox build process more consistent across platforms, we are using **Rootless Podman** for major parts of the build. The basics of using Podman are described in the [Building Redox](./podman-build.md) page. This chapter provides a detailed discussion, including tips, tricks and troubleshooting, as well as some extra detail for those who might want to leverage or improve Redox's use of Podman.

(Don't forget to read the [Build System](./build-system-reference.md) page to know our build system organization and how it works)

## Build Environment

- Environment and command line Variables, other than `ARCH`, `CONFIG_NAME` and `FILESYSTEM_CONFIG`, are not passed to the part of `make` that is done in **Podman**. You must set any other configuration variables, e.g. `REPO_BINARY`, in [.config](./configuration-settings.md#config) and not on the command line or on your environment.

- If you are building your own software to add in Redox, and you need to install additional packages using `apt-get` for the build, follow the [Adding Packages to the Build](#adding-packages-to-the-build) section.

## Installation

Most of the packages required for the build are installed in the container as part of the build process. However, some packages need to be installed on the host computer. You may also need to install an emulator such as **QEMU**. For most Linux distributions, this is done for you in the `podman_bootstrap.sh` script.

Note that the Redox filesystem parts are merged using [FUSE](https://github.com/libfuse/libfuse). `podman_bootstrap.sh` installs `libfuse` for most platforms, if it is not already included. If you have problems with the final image of Redox, verify if `libfuse` is installed and you are able to use it.

### Ubuntu

```sh
sudo apt-get install git make curl podman fuse fuse-overlayfs slirp4netns
```

### Debian

```sh
sudo apt-get install git make curl podman fuse fuse-overlayfs slirp4netns
```

### Arch Linux

```sh
sudo pacman -S --needed git make curl podman fuse3 fuse-overlayfs slirp4netns
```

### Fedora

```sh
sudo dnf install git-all make curl podman fuse3 fuse-overlayfs slirp4netns
```

### OpenSUSE

```sh
sudo zypper install git make curl podman fuse fuse-overlayfs slipr4netns
```

### Pop!_OS

```sh
sudo apt-get install git make curl podman fuse fuse-overlayfs slirp4netns
```

### FreeBSD

```sh
sudo pkg install git gmake curl fusefs-libs3 podman
```

### MacOSX

- Homebrew

```sh
sudo brew install git make curl osxfuse podman fuse-overlayfs slirp4netns
```

- MacPorts

```sh
sudo port install git gmake curl osxfuse podman
```

### NixOS

Before building Redox with NixOS, you must have configured Podman on your system. Just follow the instructions of the [NixOS wiki](https://nixos.wiki/wiki/Podman):

```nix
{ pkgs, ... }:
{
  # Enable common container config files in /etc/containers
  virtualisation.containers.enable = true;
  virtualisation = {
    podman = {
      enable = true;

      # Create a `docker` alias for podman, to use it as a drop-in replacement
      dockerCompat = true;

      # Required for containers under podman-compose to be able to talk to each other.
      defaultNetwork.settings.dns_enabled = true;
    };
  };

  # Useful other development tools
  environment.systemPackages = with pkgs; [
    dive # look into docker image layers
    podman-tui # status of containers in the terminal
    docker-compose # start group of containers for dev
    podman-compose # start group of containers for dev
  ];
}
        
```

You will then have to configure your user to be able to use Podman:
```nix
users.extraUsers.${user-name} = {
  subUidRanges = [{ startUid = 100000; count = 65536; }];
  subGidRanges = [{ startGid = 100000; count = 65536; }];
};
```

The last step is to activate the development shell:
```sh
nix develop --no-warn-dirty --command $SHELL
```


## build/container.tag

The building of the **image** is controlled by the *tag* file `build/container.tag`. If you run `make all` with `PODMAN_BUILD=1`, the file `build/container.tag` will be created after the image is built. This file tells `make` that it can skip updating the image after the first time.

Many targets in the Makefiles `mk/*.mk` include `build/container.tag` as a dependency. If the *tag* file is missing, building any of those targets may trigger an image to be created, which can take some time.

When you move to a new working directory, if you want to save a few minutes, and you are confident that your **image** is correct and your `poduser` home directory `build/podman/poduser` is valid, you can do

```sh
make container_touch
```

This will create the file `build/container.tag` without rebuilding the image. However, it will fail if the image does not exist. If it fails, just do a normal `make`, it will create the container when needed.

## Cleaning Up

To remove the **base image**, any lingering containers, `poduser`'s home directory, including the **Rust** installation, and `build/container.tag`, run:

```sh
make container_clean
```

- To verify that everything has been removed

```sh
podman ps -a
```

- Show any remaining images or containers

```sh
podman images
```

- Remove **all** images and containers. You still may need to remove `build/container.tag` if you did not do `make container_clean`.

```sh
podman system reset
```

In some rare instances, `poduser`'s home directory can have bad file permissions, and you may need to run:

```sh
sudo chown -R `id -un`:`id -gn` build/podman
```

Where `` `id -un` `` is your User ID and `` `id -gn` `` is your effective Group ID. Be sure to `make container_clean` after that.

**Note:**

- `make clean` does **not** run `make container_clean` and will **not** remove the container image.
- If you already did `make container_clean`, doing `make clean` could trigger an image build and a Rust install in the container. It invokes `cargo clean` on various components, which it must run in a container, since the build is designed to not require **Cargo** on your host machine. If you have Cargo installed on the host and in your PATH, you can use `make PODMAN_BUILD=0 clean` to clean without building a container. 

## Debugging Your Build Process

If you are developing your own components and wish to do one-time debugging to determine what library you are missing in the **Podman Build** environment, the following instructions can help. Note that your changes will not be persistent. After debugging, **you must** [Add your Libraries to the Build](#adding-packages-to-the-build). With `PODMAN_BUILD=1`, run the following command:

- This will start a `bash` shell in the **Podman** container environment, as a normal user without `root` privileges.

```sh
make container_shell
```

- Within that environment, you can build the Redox components with:

```sh
make repo
```

- If you need to change `ARCH` or `CONFIG_NAME`, run:

```sh
./build.sh -a ARCH -c CONFIG_NAME repo
```

- If you need `root` privileges, while you are **still running** the above `bash` shell, go to a separate **Terminal** or **Console** window on the host, and type:

```sh
cd ~/tryredox/redox
```

```sh
make container_su
```

You will then be running Bash with `root` privilege in the container, and you can use `apt-get` or whatever tools you need, and it will affect the environment of the user-level `container_shell` above. Do not precede the commands with `sudo` as you are already `root`. And remember that you are in an **Ubuntu** instance.

**Note**: Your changes will not persist once both shells have been exited.

Type `exit` on both shells once you have determined how to solve your problem.

## Adding Packages to the Build

This method can be used if you want to make changes/testing inside the Debian container with `make env`.

The default **Containerfile**, `podman/redox-base-containerfile`, imports all required packages for a normal Redox build.

However, you cannot easily add packages after the **base image** is created. You must add them to your own Containerfile and rebuild the container image. 

Copy `podman/redox-base-containerfile` and add to the list of packages in the initial `apt-get`.

```sh
cp podman/redox-base-containerfile podman/my-containerfile
```

```sh
nano podman/my-containerfile
```

```
...
        xxd \
        rsync \
        MY_PACKAGE \
...
```

Make sure you include the continuation character `\` at the end of each line except after the last package.


Then, edit [.config](./configuration-settings.md#config), and change the variable `CONTAINERFILE` to point to your Containerfile, e.g.

```
CONTAINERFILE?=podman/my-containerfile
```

If your Containerfile is newer than `build/container.tag`, a new **image** will be created. You can force the image to be rebuilt with `make container_clean`.

If you feel the need to have more than one image, you can change the variable `IMAGE_TAG` in `mk/podman.mk` to give the image a different name.

If you just want to install the packages temporarily, run `make env`, open a new terminal tab/windows, run `make container_su` and use `apt install` on this tab/window.

## Summary of Podman-related Make Targets, Variables and Podman Commands

- `PODMAN_BUILD` - If set to 1 in [.config](./configuration-settings.md#config), or in the environment, or on the `make` command line, much of the build process takes place in **Podman**.

- `CONTAINERFILE`-  The name of the containerfile used to build the image. This file includes the `apt-get` command that installs all the necessary packages into the image. If you need to add packages to the build, edit your own containerfile and change this variable to point to it.

- `make build/container.tag` - If no container image has been built, build one. It's not necessary to do this, it will be done when needed.

- `make container_touch` - If a container image already exists and `poduser`'s home directory is valid, but there is no *tag* file, create the *tag* file so a new image is not built.

- `make container_clean` - Remove the container image, `poduser`'s home directory and the *tag* file.

- `make container_shell` - Start an interactive Podman `bash` shell in the same environment used by `make`; for debugging the `apt-get` commands used during image build.

- `make env` - Start an interactive `bash` shell with the `prefix` tools in your PATH. Automatically determines if this should be a Podman shell or a host shell, depending on the value of `PODMAN_BUILD`.

- `make repo` or `./build.sh -a ARCH -c CONFIG repo` - Used while in a Podman shell to build all the Redox component packages. `make all` will not complete successfully, since part of the build process must take place on the host.

- `podman exec --user=0 -it CONTAINER bash` - Use this command in combination with `make container_shell` or `make env` to get `root` access to the Podman build environment, so you can temporarily add packages to the environment. `CONTAINER` is the name of the active container as shown by `podman ps`. For temporary, debugging purposes only.

- `podman system reset` - Use this command when `make container_clean` is not sufficient to solve problems caused by errors in the container image. It will remove all images, use with caution. If you are using **Podman** for any other purpose, those images will be deleted as well.

## Gory Details

If you are interested in how we are able to use your working directory for builds in **Podman**, the following configuration details may be interesting.

We are using **Rootless Podman**'s `--userns keep-id` feature. Because Podman is being run Rootless, the *container's* `root` user is actually mapped to your User ID on the host. Without the `keep-id` option, a regular user in the container maps to a phantom user outside the container. With the `keep-id` option, a user in the container that has the same User ID as your host User ID, will have the same permissions as you.

During the creation of the **base image**, Podman invokes [Buildah](https://buildah.io/) to build the image. Buildah does not allow User IDs to be shared between the host and the container in the same way that Podman does. So the base image is created without `keep-id`, then the first container created from the image, having `keep-id` enabled, triggers a remapping. Once that remapping is done, it is reused for each subsequent container.

The working directory is made available in the container by **mounting** it as a **volume**. The **Podman** option:

```sh
--volume "`pwd`":$(CONTAINER_WORKDIR):Z
```

takes the directory that `make` was started in as the host working directory, and **mounts** it at the location `$CONTAINER_WORKDIR`, normally set to `/mnt/redox`. The `:Z` at the end of the name indicates that the mounted directory should not be shared between simultaneous container instances. It is optional on some Linux distros, and not optional on others.

For our invocation of Podman, we set the PATH environment variable as an option to `podman run`. This is to avoid the need for our `make` command to run `.bashrc`, which would add extra complexity. The `ARCH`, `CONFIG_NAME` and `FILESYSTEM_CONFIG` variables are passed in the environment to allow you to override the values in `mk/config.mk` or `.config`, e.g. by setting them on your `make` command line or by using `build.sh`.

We also set `PODMAN_BUILD=0` in the environment, to ensure that the instance of `make` running in the container knows not to invoke **Podman**. This overrides the value set in `.config`.

In the **Containerfile**, we use as few `RUN` commands as possible, as **Podman** commits the image after each command. And we avoid using `ENTRYPOINT` to allow us to specify the `podman run` command as a list of arguments, rather than just a string to be processed by the entrypoint shell.

Containers in our build process are run with `--rm` to ensure the container is discarded after each use. This prevents a proliferation of used containers. However, when you use `make container_clean`, you may notice multiple items being deleted. These are the partial images created as each `RUN` command is executed while building.

Container images and container data is normally stored in the directory `$HOME/.local/share/containers/storage`. The following command removes that directory in its entirety. However, the contents of any **volume** are left alone:

```sh
podman system reset
```
