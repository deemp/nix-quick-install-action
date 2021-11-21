# Nix Quick Install Action

This GitHub Action installs [Nix](https://nixos.org/nix/) in single-user mode,
and adds almost no time at all to your workflow's running time.

The Nix installation is deterministic &ndash; for a given
release of this action the resulting Nix setup will always be identical, no
matter when you run the action.

* Supports Linux and MacOS runners

* Single-user installation (no `nix-daemon`)

* Installs in &asymp; 1 second on Linux, &asymp; 5 seconds on MacOS

* Allows selecting Nix version via the `nix_version` input

* Allows specifying `nix.conf` contents via the `nix_conf` input

## Details

The main motivation behind this action is to install Nix as quickly as possible
in your GitHub workflow. If that isn't important, you should probably use the
[Install Nix](https://github.com/marketplace/actions/install-nix) action
instead, which sets up Nix in multi-user mode (daemon mode) using the official
Nix installer.

To make this action as quick as possible, the installation is minimal: no
nix-daemon, no nix channels and no `NIX_PATH`. The nix store (`/nix/store`) is
owned by the unprivileged runner user.

The action provides you with a fully working Nix setup, but since no `NIX_PATH`
or channels are setup you need to handle this on your own. Nix Flakes is great
for this, and works perfectly with this action (see below).
[niv](https://github.com/nmattia/niv) should also work fine, but has not been
tested yet.

## Inputs

See [action.yml](action.yml) for documentation of the available inputs.
The available Nix versions are listed in the [release
notes](https://github.com/nixbuild/nix-quick-install-action/releases/latest).

## Usage

### Minimal example

The following workflow installs Nix and then just runs
`nix-build --version`:

```yaml
name: Examples
on: push
jobs:
  minimal:
    runs-on: ubuntu-latest
    steps:
      - uses: nixbuild/nix-quick-install-action@v8
      - run: nix-build --version
```

![action-minimal](examples/action-minimal.png)

### Using Nix flakes

To be able to use Nix flakes you need to specify a version of Nix that supports
it (the default Nix version, 2.4, works fine), and enable the flakes
functionality in the nix configuration:

```yaml
name: Examples
on: push
jobs:
  flakes-simple:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: nixbuild/nix-quick-install-action@v8
        with:
          nix_version: 2.4
          nix_conf: experimental-features = nix-command flakes
      - name: nix build
        run: nix build ./examples/flakes-simple
      - name: hello
        run: ./result/bin/hello
```

![action-minimal](examples/action-flakes-simple.png)

You can see the flake definition for the above example in
[examples/flakes-simple/flake.nix](examples/flakes-simple/flake.nix).

### Using Cachix

You can use the [Cachix action](https://github.com/marketplace/actions/cachix)
together with this action, just make sure you put it after this action in your
workflow.

### Using specific Nix versions locally

Locally, you can use this repository's Nix flake to install or run any of the
versions of Nix that this action supports. This is very convenient if you
quickly need to compare the behavior between different Nix versions.

Simply specify the Nix version like this:

```
$ nix shell github:nixbuild/nix-quick-install-action#nix-2_2_2 -c nix --version
nix (Nix) 2.2.2
```

You can list all available Nix versions like this:

```
$ nix flake show github:nixbuild/nix-quick-install-action
github:nixbuild/nix-quick-install-action/ecad5f06be56c8938852cc0fd599a9ade9fea616
├───apps
│   ├───x86_64-darwin
│   │   └───release: app
│   └───x86_64-linux
│       └───release: app
├───defaultApp
│   ├───x86_64-darwin: app
│   └───x86_64-linux: app
└───packages
    ├───x86_64-darwin
    │   ├───nix-2_1_3: package 'nix-2.1.3'
    │   ├───nix-2_2_2: package 'nix-2.2.2'
    │   ├───nix-2_3_10: package 'nix-2.3.10'
    │   ├───nix-2_3_12: package 'nix-2.3.12'
    │   ├───nix-2_3_14: package 'nix-2.3.14'
    │   ├───nix-2_3_15: package 'nix-2.3.15'
    │   ├───nix-2_3_7: package 'nix-2.3.7'
    │   ├───nix-2_4: package 'nix-2.4'
    │   ├───nix-2_4pre20201205_a5d85d0: package 'nix-2.4pre20201205_a5d85d0'
    │   ├───nix-2_4pre20210601_5985b8b: package 'nix-2.4pre20210601_5985b8b'
    │   ├───nix-2_4pre20210908_3c56f62: package 'nix-2.4pre20210908_3c56f62'
    │   ├───nix-archives: package 'nix-archives'
    │   └───release: package 'release'
    └───x86_64-linux
        ├───nix-2_1_3: package 'nix-2.1.3'
        ├───nix-2_2_2: package 'nix-2.2.2'
        ├───nix-2_3_10: package 'nix-2.3.10'
        ├───nix-2_3_12: package 'nix-2.3.12'
        ├───nix-2_3_14: package 'nix-2.3.14'
        ├───nix-2_3_15: package 'nix-2.3.15'
        ├───nix-2_3_7: package 'nix-2.3.7'
        ├───nix-2_4: package 'nix-2.4'
        ├───nix-2_4pre20201205_a5d85d0: package 'nix-2.4pre20201205_a5d85d0'
        ├───nix-2_4pre20210601_5985b8b: package 'nix-2.4pre20210601_5985b8b'
        ├───nix-2_4pre20210908_3c56f62: package 'nix-2.4pre20210908_3c56f62'
        ├───nix-archives: package 'nix-archives'
        └───release: package 'release'
```
