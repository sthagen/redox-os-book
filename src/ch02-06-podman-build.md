# Podman Build

To make the Redox build process more consistent across platforms, we are using **Rootless Podman** for major parts of the build. **Podman** is invoked automatically and transparently within the Makefiles.

The TL;DR version is [here](#tldr---new-or-existing-working-directory). More details are available in [Advanced Podman Build](./ch08-02-advanced-podman-build.md).

You can find out more about Podman [here](https://docs.podman.io/en/latest/Introduction.html).

(Don't forget to read [this](./ch08-06-build-system-reference.md) page to know our build system organization and how it works)

## Disabling Podman Build

By default, Podman Build is disabled. The variable `PODMAN_BUILD` in `mk/config.mk` defaults to zero, so that **Podman** will not be invoked. If you find that it is enabled but you want it disabled, set `PODMAN_BUILD?=0` in [.config](./ch02-07-configuration-settings.md#config), and ensure it is not set in your environment, `unset PODMAN_BUILD`.

## Podman Build Overview

**Podman** is a **user-space manager** that creates **containers** to execute a Linux distribution **image**. In our case, we are creating an **Ubuntu** image, with a **Rust** installation and all the packages needed to build the system and programs.

The build process is performed in your normal working directory, e.g. `~/tryredox/redox`. Compilation of the Redox components is performed in the container, but the final Redox image (`build/$ARCH/$CONFIG/harddrive.img` or `build/$ARCH/$CONFIG/livedisk.iso`) is constructed using [FUSE](https://github.com/libfuse/libfuse) running directly on your host machine.

Setting `PODMAN_BUILD` to 1 in [.config](./ch02-07-configuration-settings.md#config), on the `make` command line (e.g. `make PODMAN_BUILD=1 all`) or in the environment (e.g. `export PODMAN_BUILD=1; make all`) will enable Podman.

First, a **base image** called `redox_base` will be constructed, with all the necessary packages for the build system. A "home" directory will also be created in `build/podman`. This is the home directory of your container alter ego, `poduser`. It will contain the `rustup` install, and the `.bashrc`. This takes some time, but is only done when necessary. The *tag* file [build/container.tag](./ch08-02-advanced-podman-build.md#buildcontainertag) is also created at this time to prevent unnecessary image builds.

Then, various `make` commands are executed in **containers** built from the **base image**. The files are constructed in your working directory tree, just as they would for a non-Podman build. In fact, if all necessary packages are installed on your host system, you can switch Podman on and off relatively seamlessly, although there is no benefit of doing so.

The build process is using **Podman**'s `keep-id` feature, which allows your regular User ID to be mapped to `poduser` in the container. The first time a container is built, it takes some time to set up this mapping. After the first container is built, new containers can be built almost instantly.

## TL;DR - [New](#new-working-directory) or [Existing](#existing-working-directory) Working Directory

### New Working Directory 

If you have already read the [Building Redox](./ch02-05-building-redox.md) instructions, but you wish to use **Podman Build**, follow these steps.

- Make sure you have the `curl` command. e.g. for Pop!_OS/Ubuntu/Debian:

```sh
which curl || sudo apt-get install curl 
```

- Make a directory, get a copy of `podman_bootstrap.sh` and run it. This will clone the repository and install **Podman**.

```sh
mkdir -p ~/tryredox
```

```sh
cd ~/tryredox
```

```sh
curl -sf https://gitlab.redox-os.org/redox-os/redox/raw/master/podman_bootstrap.sh -o podman_bootstrap.sh
```

```sh
time bash -e podman_bootstrap.sh
```

- Change to the `redox` directory.

```sh
cd ~/tryredox/redox
```

- Verify if the file [.config](./ch02-07-configuration-settings.md#config) was created in the `redox` base directory, and contains the line `PODMAN_BUILD?=1`.
- Build the system. This will take some time.

```sh
time make all
```

### Existing Working Directory

If you already have the source tree, you can use these steps.

- Change to your working directory and get the updates to the build files.

```sh
cd ~/tryredox/redox
```

```sh
make pull
```

- Install **Podman**. Many Linux distributions require additional packages. Check the [Minimum Installation](./ch08-02-advanced-podman-build.md#minimum-installation) instructions to see what is needed for your distribution. Or, run the following in your `redox` base` directory:

```sh
./podman_bootstrap.sh -d
```

- Set `PODMAN_BUILD` to 1 and run `make all`. The first container setup can take 15 minutes or more, but it is comparable in speed to native build after that.

```sh
export PODMAN_BUILD=1
```

```sh
make all
```

- Open the QEMU to see if the image was properly created

```sh
make qemu
```

To ensure `PODMAN_BUILD` is properly set for future builds, edit [.config](./ch02-07-configuration-settings.md#config) in your base `redox` directory and change its value.

```sh
nano .config
```

```
PODMAN_BUILD?=1
```

