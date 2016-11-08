## Requirements ##

- git >= 1.8.2

## Installing ##

On all operating systems, once git-lfs is installed, `git lfs install` must be run. Each user that intends to use git lfs must run this command, but they only ever need to run it once. git-lfs can be disabled by running `git lfs uninstall`, in which case that user would have to run `git lfs install` again, before git-lfs features work again.

Some users may wish to only enable git-lfs on specific repositories instead of always having it on for all of the repositories. Instead of running `git lfs install` and enabling git-lfs for that entire user, `git lfs install --local` can be used instead on a per repository basis. 

Note: If git-lfs is installed without `--local`, then `git lfs uninstall --local` would not disable it for a specific repository.

An additional option of `--skip-smudge` can be added to skip automatic downloading of objects on clone or pull. This requires a manual `git lfs pull` every time a new commit is checked out on your repository. This is more useful for cases where you don't always want to download/checkout every large file.

### Debian ###

- Debian 7 Wheezy and similar needs to have the backports repo installed to get git >= 1.8.2

    - `echo 'deb http://http.debian.net/debian wheezy-backports main' > /etc/apt/sources.list.d/wheezy-backports-main.list`

1. `curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash`
2. `sudo apt-get install git-lfs`
3. `git lfs install`

### Mac OSX ###

1. You may need to `brew update` to get all the new formulas
2. `brew install git-lfs`
3. `git lfs install`

### RHEL/CentOS ###

1. Install git >= 1.8.2

    - Recommended method for RHEL/CentOS 5 and 7 (not 6!)

        1. Install the epel repo [link](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F) (For CentOS it's just `sudo yum install epel-release`)
        2. `sudo yum install git-lfs`

    - Recommended method for RHEL/CentOS 6

        1. Install the IUS Community repo. `curl -s https://setup.ius.io/ | sudo bash` or [here](https://ius.io/GettingStarted/)
        2. `sudo yum install git2u`

    - You can also build git from source and install it. If you do that, you will need to either manually download the the git-lfs rpm and install it with `rpm -i --nodeps git-lfs*.rpm`, or just use the [Other](#Other) instructions. The only other advanced way to fool yum is to create and install a fake/real git rpm to satisfy the git >= 1.8.2 requirement. 

- To install the git-lfs repo, run `curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash` from [here](https://packagecloud.io/github/git-lfs/install)
- `sudo yum install git-lfs`
- `git lfs install`

### Ubuntu ###

- Similar to Debian 7, Ubuntu 12 and similar Wheezy versions need to have a PPA repo installed to get git >= 1.8.2

    1. `sudo apt-get install python-software-properties` to install add-apt-repository
    2. `sudo add-apt-repository ppa:git-core/ppa`
    3. The curl script below calls apt-get update, if you aren't using it, don't forget to call `apt-get update` before installing git-lfs.

1. `curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash`
2. `sudo apt-get install git-lfs`
3. `git lfs install`

### Windows ###

1. Download the windows installer from [here](https://github.com/github/git-lfs/releases)
2. Run the windows installer 
3. Start a command prompt/or git for windows prompt and run `git lfs install`

### Docker Recipes ###

For Debian Distros, you can use

```dockerfile
RUN build_deps="curl ca-certificates" && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ${build_deps} && \
    curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends git-lfs && \
    git lfs install && \
    DEBIAN_FRONTEND=noninteractive apt-get purge -y --auto-remove ${build_deps} && \
    rm -r /var/lib/apt/lists/*
```

## Other ##

To install on any supported operating system, manually install git-lfs with no man pages.

Only one file is required for git-lfs, the `git-lfs` binary. i386 and x86_64 versions are available [here](https://github.com/github/git-lfs/releases) for FreeBSD, Linux, Mac and Windows. Currently, linux arm6 must be compiled from source

1. Install git version 1.8.2 or newer
2. Download and put the git-lfs (.exe for windows) in your path, and `git lfs` commands start working, as long as both git and git-lfs are in your path.

## Source ##

1. Get go. Either use your OS repo or get it [here](https://golang.org/dl/). For best results, use latest stable go to get all the patches
2. `git clone https://github.com/github/git-lfs.git`
3. In the git-lfs directory, run `./script/bootstrap`
4. The git-lfs binary should appear in the `./bin` directory

Alternatively, you can use go to

1. `go get github.com/github/git-lfs`
2. `$GOPATH/bin/`
3. Edit/checkout `$GOPATH/src/github.com/github/git-lfs` and run `go build github.com/github/git-lfs` if needed