devbox-play: Experiments with devbox/nix
========================================

Working from the [source code README][gh].

General:
- The curl magic installs a `devbox` bootstrap binary in `/usr/local/bin/`;
  via `sudo`. This `devbox` second-stage bootstrap script can just as well
  go in `~/.local/bin/`, though the install script downloaded by curl is
  hard-coded to use sudo and `/usr/local/bin/`.
- `devbox init` creates only a fresh `devbox.json` in the current working
  directory (even if it's not the root of the repo); this becomes the
  "project dir" found even if you subsequently run devbox commands below
  that directory.
- `devbox add asl@latest` will create `.devbox/` in the project dir;
  this is ignored by git in a clever way (see below).

Notes:
- Set `DEVBOX_DEBUG=1` in the environment to get a verbose log of what
  `devbox` is doing, and any errors that result.
- Devbox does not work with Nix 2.3.7 due to lack of the `--impure` flag.

Interesting points:
- The `$PROJECT_DIR/.devbox/` directory is ignored without having to
  commit anything to get by creating a `.gitignore` _under_ that directory
  containing `*` and `.*`.



<!-------------------------------------------------------------------->
[gh]: https://github.com/jetpack-io/devbox?tab=readme-ov-file#quickstart-fast-deterministic-shell

