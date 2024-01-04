devbox-play: Experiments with devbox/nix
========================================

Working from the [source code README][gh].

Installation:
- The curl magic installs a `devbox` bootstrap binary in `/usr/local/bin/`;
  via `sudo`. This `devbox` second-stage bootstrap script can just as well
  go in `~/.local/bin/`, though the install script downloaded by curl is
  hard-coded to use sudo and `/usr/local/bin/`.
- Our `Test` script tweaks the above to do a sudo-free install to
  `~/.local/bin/`.
- `devbox add` will, if `/nix/` does not exist, use `sudo` to create the `/nix/`
  directory and change ownership to the current user (probably), and then
  attept to a single-user install of Nix 2.18 from `releases.nixos.org`,
  giving you a chance to abort first. This uses
  `curl -L https://releases.nixos.org/nix/nix-2.18.1/install | sh -s`.

General:
- `devbox init` creates only a fresh `devbox.json` in the current working
  directory (even if it's not the root of the repo); this becomes the
  "project dir" found even if you subsequently run devbox commands below
  that directory.
- `devbox add asl@latest` will create `.devbox/` in the project dir;
  this is ignored by git in a clever way (see below).

Notes:
- Set `DEVBOX_DEBUG=1` in the environment to get a verbose log of what
  `devbox` is doing, and any errors that result.
- Devbox does not work with:
  - Nix 2.3.7 (Debian 11) due to lack of the `--impure` flag.
  - Nix 2.8.0 (Debian 12), due to lack of `--priority` flag on
    `nix profile install`
  - Nix 2.18 seems to be the desired version; see above.
  - 2.18 install fails on Debian 12; see below.

Interesting points:
- The `$PROJECT_DIR/.devbox/` directory is ignored without having to
  commit anything to get by creating a `.gitignore` _under_ that directory
  containing `*` and `.*`.


Errors and Workarounds
----------------------

### Debian 12 Nix 2.18 Install Failure

`devbox add asl@latest` without a `/nix/` directory makes Devbox attempt
to install nix (per above), which fails with:

    Installing nix with: curl -L https://releases.nixos.org/nix/nix-2.18.1/install | sh -s
    This may require sudo access.
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  4052  100  4052    0     0  32017      0 --:--:-- --:--:-- --:--:-- 31905
    downloading Nix 2.18.1 binary tarball for x86_64-linux from 'https://releases.nixos.org/nix/nix-2.18.1/nix-2.18.1-x86_64-linux.tar.xz' to '/tmp/nix-binary-tarball-unpack.SqZ3zIE1Sv'...
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100 20.6M  100 20.6M    0     0  7362k      0  0:00:02  0:00:02 --:--:-- 7365k
    Note: a multi-user installation is possible. See https://nixos.org/manual/nix/stable/installation/installing-binary.html#multi-user-installation
    performing a single-user installation of Nix...
    directory /nix does not exist; creating it by running 'mkdir -m 0755 /nix && chown cjs /nix' using sudo
    copying Nix to /nix/store...

    installing 'nix-2.18.1'
    building '/nix/store/hzx954rnda4k348rb4wvqbdzp9z0ynjy-user-environment.drv'...
    error: opening lock file '/nix/var/nix/profiles/per-user/cjs/profile.lock': No such file or directory
    /tmp/nix-binary-tarball-unpack.SqZ3zIE1Sv/unpack/nix-2.18.1-x86_64-linux/install: unable to install Nix into your default profile
    Error: exit status 1

The problem may be that the `…/cjs/` directory above does not exist. It
can be created with

    mkdir /nix/var/nix/profiles/per-user/cjs

However, re-running Devbox at this point won't work because it thinks that
Nix is installed but it is not. So you must manuall re-run the Nix install:

    curl -L https://releases.nixos.org/nix/nix-2.18.1/install | sh -s

But the actual problem turns out to be `~/.nix-*` files/dirs left behind
from a previous Nix install, `rm -rf ~/.nix-*` fixes the install to
work the first time.


<!-------------------------------------------------------------------->
[gh]: https://github.com/jetpack-io/devbox?tab=readme-ov-file#quickstart-fast-deterministic-shell

