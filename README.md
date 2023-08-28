# git-remote-rclone-sync

A [git remote helper](https://git-scm.com/docs/gitremote-helpers) which uses [rclone](https://rclone.org/) to access git repositories.  This means you can store your repo using one of the [dozens](https://rclone.org/overview/) of cloud providers and protocols supported by rclone, optionally taking advantage of rclone features like transparent [encryption](https://rclone.org/crypt/), [compression](https://rclone.org/compress/) and [mirroring](https://rclone.org/union/).


### Prerequisites

- Written and tested on Linux, but should work on Mac OS, BSD or any Unix-like OS.  May or may not work under git-bash/Cygwin on Windows, but not having a Windows machine I can't test this.
- Written in [bash](https://www.gnu.org/software/bash/) - should even work on the ancient bash in Mac OS, but this hasn't been tested.
- Requires [rclone](https://rclone.org/) and [git](https://git-scm.com/) obviously - tested with rclone 1.53.3 and git 2.20.1, but any reasonably-recent version should work.


### Getting started

Download the script and put it in your PATH:

```
wget -O ./git-remote-rclone-sync https://raw.githubusercontent.com/pobrelkey/misc-scripts/master/git-remote-rclone-sync
sudo install ./git-remote-rclone-sync /usr/local/bin
```

Set up an rclone remote pointing to where your remote repository is stored - either by manually editing your `rclone.conf` file, or by running `rclone config` and following the prompts.

Now you can clone an existing repository:

```
git clone rclone-sync::name-of-rclone-remote:path/too/repos.git
```

Or you can add an additional remote to an existing working branch:

```
git remote add some-remote rclone-sync::name-of-rclone-remote:path/too/repos.git
git push some-remote
```

Pulls from and pushes to the remote should require no different/additional commands than any other remote.

If a git repository doesn't already exist at the remote location, pulls and pushes will fail.  The remote helper contains a built-in convenience script you can use to create a minimal bare Git repository and copy it to the remote location:

```
git-remote-rclone-sync --init name-of-rclone-remote:path/too/repos.git "optional brief description"
```


### Notes and limitations

**Concurrent access:** This tool does not attempt to detect or remedy situations where another process/user is writing to the remote repository at the same time other users are attemping access.  (This would be impossible to implement across all rclone remote types - particularly so for cloud storage providers with "eventually consistent" semantics.)  To avoid corruption, only use this tool in use cases where concurrent access is unlikely, or implement your own means to ensure only one user accesses the remote repository at a time, using a "check-in [token](https://en.wikipedia.org/wiki/Token_(railway_signalling))" or other similar mechanism.

**Disk usage:**  This tool keeps a full copy of the remote repository in the `.git` directory of your working copy, which is kept in sync with the remote using `rclone sync`.  So disk usage may be significant, particularly for repositories with long histories, large binary objects, etc.

**Mount mode:**  If local disk space is at a premium, you can instead access your remote repositories using rclone's built-in [FUSE mounting](https://rclone.org/commands/rclone_mount/) functionality, thereby trading lower local disk usage for more intensive network access.  To do this, create a copy of or symlink to this script named `git-remote-rclone-mount`, then use remote repository URLs which start with `rclone-mount` instead of `rclone-sync` - for example, `rclone-mount::name-of-rclone-remote:/path/to/repos.git`.  Mount mode is likely to only work on Linux and perhaps BSD variants - and then only where your kernel and your user permissions allow FUSE mounts - but not on Mac OS (where the command to unmount a FUSE volume is different) or Windows (where rclone's WinFSP mounts work differently).  Also, note that the rclone developer [warns](https://rclone.org/commands/rclone_mount/#rclone-mount-vs-rclone-sync-copy) that `rclone mount` is less reliable than `rclone sync`, so use mount mode at your own risk.

**Repository format:**  This tool expects the git repository as exposed by rclone to be stored in the ordinary Git format, much as one might create on the local disk using `git init --bare`, rather than some other specialized format.  No attempt is made to re-pack, consolidate, compress or otherwise optimize the remote repository to reduce the number of remote objects, amount of storage consumed or number of access calls required.  (This is unlikely to be an issue for small repositories.)  A by-product of this is that if the remote repository is stored on a remote storage provider which lacks the concept of empty directories (such as Amazon S3), this tool will fail to recognize it as a git repository unless it contains at least one commit.  (`git-remote-rclone-sync --init` creates a single commit, containing a stub `README.md` file, before copying the repository to the remote end.)

**Encrypted/custom rclone config:**  Encrypted rclone configuration is supported; git's [credential propmting/caching mechanism](https://git-scm.com/docs/gitcredentials) will be used to obtain the config file password.  You can specify a different `rclone.conf` for a particular working directory by writing it to `.git/rclone-sync/rclone.conf`.

**Name:**  This script would have just been called `git-remote-rclone`, but this name is already taken on [Github](https://github.com/datalad/git-remote-rclone) and [PyPI](https://pypi.org/project/git-remote-rclone/) by another similar tool which uses a non-standard remote repository format.


### License

Licensed under the MIT License - see [LICENSE](LICENSE) for details. 
