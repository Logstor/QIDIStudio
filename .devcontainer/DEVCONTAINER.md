# QIDIStudio development container

This configuration gives Zed a minimal Debian build environment without
installing the QIDIStudio toolchain or libraries on the host. The host still
needs Zed and either Docker or Podman because Zed uses the host container
runtime to create the development container.

The container is a development environment, not a replacement for the existing
release/container images. It bind-mounts the checkout instead of copying it,
does not compile during image creation, and does not launch QIDIStudio as its
entrypoint. Its build context is only `.devcontainer/`, so an existing large
`build/` or `deps/build/` tree is not sent to the container builder.

## Install in the repository

Copy `.devcontainer/` and `.zed/` into the root of the QIDIStudio checkout.
Commit them if the environment should be shared by all contributors.

If using Podman, set this in Zed's user `settings.json`:

```json
{
  "use_podman": true
}
```

Open the checkout in Zed and choose **Open in Container** when prompted. It is
also available from **Project: Open Remote** in the command palette.

Zed currently does not automatically rebuild a container after
`devcontainer.json` changes. Stop the old container and open the project in the
container again after editing `.devcontainer/`.

## Build the open-source configuration

The devcontainer sets `QDT_RELEASE_TO_PUBLIC=0`, explicitly disabling the
proprietary QIDI cloud/account layer. `version.inc` also auto-detects a missing
`src/slic3r/QIDI/QIDINetwork.cpp`, but the environment variable makes the
development build's intent unambiguous.

From Zed's terminal, first initialize submodules if the checkout has not
already done so:

```bash
git submodule update --init --recursive
```

Build the third-party dependencies once, then QIDIStudio:

```bash
./BuildLinux.sh -d
./BuildLinux.sh -s
```

For a debug build:

```bash
./BuildLinux.sh -bd
./BuildLinux.sh -bs
```

The image uses `debian:bookworm-slim`, `apt --no-install-recommends`, Debian's
built-in `C.UTF-8` locale, and a single cleaned package layer. Its package list
tracks `linux.d/debian`; the only editor-specific addition is clangd. Debuggers,
formatters, GUI test harnesses, Docker clients, and convenience utilities are
not installed by default.

Normally there is no reason to run `BuildLinux.sh -u`. If `linux.d/debian`
gains a new system dependency before the Dockerfile is updated, run:

```bash
sudo ./BuildLinux.sh -u
```

That installation remains inside the container.

`CMAKE_EXPORT_COMPILE_COMMANDS=ON` and the project Zed settings point clangd at
`build/compile_commands.json`, so code navigation should become accurate after
the first CMake configure.

## Optional debugging and headless GUI testing

Install optional tools inside the disposable container only when needed:

```bash
sudo apt-get update
sudo apt-get install --no-install-recommends gdb xvfb
xvfb-run -a ./build/package/bin/qidi-studio
```

The baseline deliberately does not mount the host's home directory, display
sockets, devices, or Docker socket, and it does not use privileged mode. This
is safer than the existing `DockerRun.sh`, whose job is to run the packaged GUI
rather than provide an editor environment.

## Optional Docker access

You do not need nested Docker for normal development: run `BuildLinux.sh`
directly in the devcontainer. If you must run `DockerBuild.sh` from its terminal,
use a Docker-outside-of-Docker setup that mounts the host Docker socket and
installs a Docker CLI in the image. Treat that as host-level access: possession
of the Docker socket is effectively root-equivalent on the host. Keeping it out
of the default configuration avoids silently weakening the isolation.

## Resources

The build script requires more than 10 GiB of available memory and 10 GiB of
free disk unless invoked with `-r`. Building the dependencies takes
considerably more disk than that in practice. Build products stay under the
bind-mounted checkout (`build/` and `deps/build/`) and survive container
recreation; only installed packages and other container filesystem changes are
discarded when the container is rebuilt.
