devbox-play: Experiments with devbox/nix
========================================

Working from the [source code README][gh].

__WARNING:__ After `devbox add …` you should exit and restart any `devbox
shell` instances so that they get the new configuration/plugins specific to
the package. (`refresh` in the shell will not work (we guess) because it
refreshes only env vars, it does not re-run package hooks or `init_hook`.)

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
- `devbox add asl@latest` will create `.devbox/` in the project dir; this
  is ignored by git in a clever way (see below). It also updates
  `devbox.json` to change `"packages": [],` to `"packages":
  ["asl@latest"],`, and adds `devbox.lock` to add the details and Nix IDs
  of the package. Also seems to create the `.devbox/` subdir
- When using a Devbox-installed Python, there is special support to
  use a project-local create a `venv` virtual environment (in $VENV_DIR, default
  `.devbox/virtenv/python/.venv`) for the project. This is not
  automatically activated.
  - The automatic creation happens not when you `devbox add python` but
    only after you exit and `devbox shell` again. That runs the helper
    file `.devbox/virtenv/python/bin/venvShellHook.sh` which creates it
    in $VENV_DIR. (This cannot be run stand-alone.)
  - To automatically activate, ; add `source $VENV_DIR/bin/activate` to the
    `init_hook` of `devbox.json`. (You can also set $VENV_DIR there.)
  - This info can be shown with `devbox info python`.
  - Various things can cause system Python stuff to leak into the devbox
    environment if you're not using a Python venv.

Common commands:
- `devbox install PKG[@VER]`
- `devbox info PKG`: Shows version and other info, including special
  Devbox handling of e.g. Python.
- `devbox shell`: Start a subshell with the devbox environment for the
  project found from the CWD. Has the usual massive slew of long
  `/nix/store/…` paths at the front of $PATH. Use `refresh` within the
  shell to do some sort of environment update (but does not re-run
  `init_hook` or package hooks).
- `devbox shellenv`: Prints `export` commands to update the environment for
  the Nix profile and all its various packages, adds a `refresh`
  alias to re-run the update, and executes `hash -r`.
- `devbox run`: Run a single command in the project environment.
- `devbox setup nix`: ???

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

However, after that it comes up with:

    Nix installed successfully. Devbox is ready to use!
    Error: exec: "nix": executable file not found in $PATH

Maybe it's not re-running whatever extra `.bash_profile` stuff that the
Nix install added? I.e., this (line split for clarity):

    if [ -e /home/cjs/.nix-profile/etc/profile.d/nix.sh ]; then
      . /home/cjs/.nix-profile/etc/profile.d/nix.sh;
    fi # added by Nix installer

Note that this does nothing unless both `$HOME` and `$USER` are set in
the environment. The `dent` container I'm using for some reason doesn't
have `$USER` set. This needs investigation.


<!-------------------------------------------------------------------->
[gh]: https://github.com/jetpack-io/devbox?tab=readme-ov-file#quickstart-fast-deterministic-shell

