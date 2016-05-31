# Git LFS Tutorial 

First things first, install git 1.8.2 or newer and then install git-lfs (See [here](Installation))

## Create a new repo ##

    git init .
    echo Hello World > README.md
    git add README.md
    git commit -m "Initial commit"

Let's create some more files

    ls > foo.txt
    ls > bar.txt
    ls > cat.bin
    ls > dog.bin

And add them to git

    git add foo.txt bar.txt cat.bin dog.bin

Now we are going to decide that `*.bin` files are large files that we want git-lfs to track them. Tracking means that in subsequent commits, these files will now be LFS files. 

*Note: This does NOT mean that versions of the files in previous commits will be converted (involving a process commonly known as rewriting history). This is a different process (See [BFG](#using-bfg-to-migrate-repo))*

We do this by setting a `track` pattern, using the `git lfs track` command.

    git lfs track '*.bin'

This tells git-lfs to track all files matching the `*.bin` pattern. The quotes around the `*.bin` are there to prevent the shell from expanding `*.bin`, else you would end up only tracking `cat.bin dog.bin`, and no other `bin` in the future.

To see a list of all patterns currently being track by git-lfs, run `git lfs track` with no arguments

```
Listing tracked paths
    *.bin (.gitattributes)
```

To see the list of files being track by git-lfs, run `git lfs ls-files`. You will see that this list is currently empty. This is because technically the file isn't an lfs object until after you commit it. 

Next, you need to add `.gitattributes` to your git repository. `git lfs track` stores the tracked files patterns in `.gitattributes`. This way when the repo is cloned, the track files patterns are preserved

    git add .gitattributes

Now `git status` should look like this.

```
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitattributes
        new file:   bar.txt
        new file:   cat.bin
        new file:   dog.bin
        new file:   foo.txt
```

Finally, commit the new files

    git commit -m "Added files"

Now, when you run `git lfs ls-files`, you will see the list of tracked files

```
f05131d24d * cat.bin
7db207c488 * dog.bin
```

## LFS URL ##

When using a git server that supports lfs, the lfs url defaults upon clone/adding a remote. However, sometimes the LFS server and git server are two separate services (such as the [lfs-test-server](https://github.com/github/lfs-test-server). 

To see the lfs url, run `git lfs env`

```
git-lfs/1.1.1 (GitHub; windows amd64; go 1.5.3; git 7de0397)
git version 2.6.4.windows.1

Endpoint=https://example.com/path/myrepo.git/info/lfs (auth=none)
Endpoint (upstream)=example.com/path/myrepo.git/info/lfs (auth=none)
LocalWorkingDir=C:\src\myrepo
LocalGitDir=C:\src\myrepo\.git
LocalGitStorageDir=C:\src\myrepo\.git
LocalMediaDir=C:\src\myrepo\.git\lfs\objects
TempDir=C:\src\myrepo\.git\lfs\tmp
ConcurrentTransfers=3
BatchTransfer=true
git config filter.lfs.smudge = "git-lfs smudge %f"
git config filter.lfs.clean = "git-lfs clean %f"
```

The endpoint identifies the server URL the lfs objects are sent to. When using an external server like [lfs-test-server](https://github.com/github/lfs-test-server), you need to override the default URL to point to your external URL. This can be done using `git config lfs.url = "https://my_other_server.example.com/foo/bar/info/lfs"`, however this modifies the `.git/config` file, which is a local git file, and cannot be added to your git repository. To get around this, git-lfs will also use a custom `.lfsconfig` file.

    git config -f .lfsconfig lfs.url https://my_other_server.example.com/foo/bar/info/lfs
    git add .lfsconfig

Now, `git lfs env` shows: 

```
git-lfs/1.1.1 (GitHub; windows amd64; go 1.5.3; git 7de0397)
git version 2.6.4.windows.1

Endpoint=https://my_other_server.example.com/foo/bar/info/lfs (auth=none)
...
```

*Note: only `http://`/`https://` endpoints are supported by the git-lfs client. There is currently no direct `ssh://` or `file://`. This means you can NOT have a local file lfs repo like you can for git. You CAN run a local lfs-test-server, if you want to run everything locally. `http://` should never be used over the internet, as your password will not be securely encrypted. Only use `https://`*

## Credential caching/storage ##

Since git-lfs only supports http/https, git will need to authenticate over http/https when pushing files, even if you are using ssh/git protocol for git. Without the credential helper, you will be asked to enter your username and password for EVERY connection, which is pretty unusable. To get around this, git [credential](https://git-scm.com/docs/git-credential-cache) helpers will help handling passwords for you.

- Linux - [Cacher](https://help.github.com/articles/caching-your-github-password-in-git/#platform-linux) will store the passwords in memory so you only have to enter the password once until the session timesout (usually 900 seconds)
- Mac - [osxkeychain](https://help.github.com/articles/caching-your-github-password-in-git/#platform-mac) to use the osx keychain for your password.
- Windows - [wincred](https://help.github.com/articles/caching-your-github-password-in-git/#platform-windows) is the default built in for windows and should be good for most uses. There is another helper you can use called the [Git Credential Manager for Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) aka wincred, the successor of the old winstore helper. It has some extra authentication support for GitHub and VSTS.
- Store - The final option is to store your username and password using the [store](https://git-scm.com/docs/git-credential-store) helper. This should only be used when no other option is available (such as automatic private builds on trusted server) 

## Adding git-lfs to a pre-exiting repo  ##

Let's create pretend repo to understand how converting to git-lfs works

```
git init .
touch README.md
git add README.md
git commit -m "initial"
git tag one
echo hi > plain.txt
ls > foo.bin
git add plain.txt foo.bin
git commit -m "Added some files"
git tag two
echo bye > plain.txt
ls > bar.bin
ls > foo.bin
git add plain.txt foo.bin bar.bin
git commit -m "Updated and added another file"
git tag three
echo bye >> plain.txt
ls > foo.bin
git add plain.txt foo.bin
git commit -m "Updated some more"
git tag four
```

Now lets decide we want `*.bin` files to be turned into lfs objects. 

```
git lfs track '*.bin'
git add .gitattributes
git commit -m "Now tracking bin files"
git tag not_working
```

Just tracking files does NOT convert them to lfs. They are part of history already, the only way to convert old files is to rewrite history (See [BFG}(#using-bfg-to-migrate-repo)).

Next we can try converting the latest version of history only to LFS. Since `*.bin` are currently tracked, we just need to add the files back after removing them. We will remove it from git without deleting the file (aka remove cache), and then add it back

```
git rm --cached *.bin
git add *.bin
git commit -m "Convert last commit to LFS"
```

Now, `git lfs ls-files` shows:

```
4665a5ea42 * bar.bin
4665a5ea42 * foo.bin
```

We can also see that the `foo.bin` is an lfs file by `git show HEAD:foo.bin`

```
version https://git-lfs.github.com/spec/v1
oid sha256:4665a5ea423c2713d436b5ee50593a9640e0018c1550b5a0002f74190d6caea8
size 36
```

But when we look at the previous histories, we see that they are still not LFS tracked. `git show not_working:foo.bin`

```
bar.bin
foo.bin
plain.txt
README.md
```

It's important to understand that tracking an LFS file does not remove it from a previous version of history. If you create a repo without LFS before, and put hundreds of MB in there, after all these steps, the old version still has all the large files in git, not in LFS. There is a discussion on how to rewrite history [here](https://github.com/github/git-lfs/issues/326), but the general consensus is that using [bfg-repo-cleaner](https://github.com/rtyley/bfg-repo-cleaner/releases) is the best path forward. See the next section on how to do this.

## Using BFG to migrate repo ##

# Don't use this, Use https://github.com/bozaro/git-lfs-migrate instead

1. Download and install `java >= 6`
2. Download `bfg` [here](https://rtyley.github.io/bfg-repo-cleaner/#download) and put it somewhere
3. `cd {git repo}`
4. `java -jar {bfg_dir}/bfg-1.12.8.jar --convert-to-git-lfs '*.dll' --no-blob-protection`
5. `java -jar {bfg_dir}/bfg-1.12.8.jar --convert-to-git-lfs '*.exe' --no-blob-protection`
6. And so on, one pattern at a time. This is the current recommendation.
7. `.gitattributes` are not generated through out history... So the best bet is to start tracking on the newest version only. This WILL go bad if you go to a previous version of history, branch off, commit, as LFS track files will not know to track at that point. I guess this would have to be fixed via ` git-filter-branch`
8. `git lfs track '*.dll'` '*.exe'`
10. For all the patterns:
11. `git add .gitattributes`
12. `git commit -m "Added .gitattributes for lfs tracking"`
12. `git reflog expire --expire=now --all`
13. `git gc --prune=now`
13. Final step, `git push origin --all -f` and `git push origin --tags -f`. The `-f` is the important point of no return. This pushes a DIFFERENT repo now. Anyone else out there who has a clone, will have the broken clone, and will need to re-clone the new better lfs version.
14. Don't be alarmed that `git status` shows all your files as different. This is a common git issue where the index is confused. It can be fixed by.... Well, not sure the best way. Throw away the clone, and clone again is one way, checkout initial commit and then go back is another (this only works if none of the affected files exist in the initial commit). Both of these are BAD solutions. The best way is _____?

## Pulling and cloning ##

TODO:

* Push
* Pushing missing files `git lfs push origin master --all`
* Cloning
* Pulling as clone, pulling after clone, etc...
* Pulling missed files `git checkout .`, `git lfs ls-files` shows `-` for missing pulled files(smudged), and `*` for pulled files(cleaned)
* Pulling all LFS objects (to conveniently work offline) `git lfs fetch --all`, there is also a `--recent` flag to only fetch the last few days. See `git lfs fetch --help` for more details

## LFS pointer files (Advanced) ##

Git LFS stores a pointer file in the git repo in lieu of the real large file. The pointer is swapped out for the real file at checkout (using `smudge` and `clean`). Normally, you would never see it, but for the curious, you can view the pointer file by `git show HEAD:{filename}`, for example:

`cat cat.bin`

    bar.txt
    cat.bin
    foo.txt
    README.md

`git show HEAD:cat.bin`

    version https://git-lfs.github.com/spec/v1
    oid sha256:f05131d24da96daa6a6712c5b9d368c81eeaea5dc7d0b6c7bec7d03ccf021b4a
    size 34

You can also create any pointer you want, `cat dog.bin | git lfs clean`

    version https://git-lfs.github.com/spec/v1
    oid sha256:7db207c488a5957f0b88aec97489a60e73f2b8d310586c2502f3388af7f76091
    size 42

Running `git lfs clean` like this, also creates a new lfs object for every clean. Not sure why you'd want to. For example: `echo hi | git lfs clean`:

    version https://git-lfs.github.com/spec/v1
    oid sha256:98ea6e4f216f2fb4b69fff9b3a44842c38686ca685f3f55dc48c5d3fb1107be4
    size 3

Now, `ls .git/lfs/objects/98/ea` shows:

    98ea6e4f216f2fb4b69fff9b3a44842c38686ca685f3f55dc48c5d3fb1107be4

You can also generate a pointer using `git lfs pointer`, but does not create lfs objects.