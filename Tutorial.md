# Git LFS Tutorial 

First things first, install git 1.8.2 or newer and then install git-lfs (See [here](Installation))

## Create a new repo ##

    git init .
    echo Hello World > README.md
    git add README.md
    git commit -m "Initial commit"

Let's create some files

    ls > foo.txt
    ls > bar.txt
    ls > cat.bin
    ls > dog.bin

Now we are going to decide that `*.bin` files are large files that we want git-lfs to track, and all other files are not. We do this by setting a `track` pattern, using the `git lfs track` command.

   git lfs track '*.bin'

The quotes around the `*.bin` are there to prevent the shell from expanding `*.bin`, else you would end up with `cat.bin dog.bin`

* go over .gitattributes
* go over .lfsconfig files
  * http only, not git protocol
* Listing lfs tracked files `git lfs ls-files
* Push
* Pushing missing files `git lfs push origin master --all`
* cloning
* pulling as clone, pulling after clone, etc...
* pulling missed files `git checkout .`, `git lfs ls-files` shows `-` for missing pulled files(smudged), and `*` for pulled files(cleaned)
* Credential caching/storage

## Looking at the point

`git show HEAD:{filename}`

## Adding git-lfs to a pre-exiting repo  ##

TODO

## Using BFG to migrate repo ##

1. Download and install `java >= 6`
2. Download bfg [here](https://rtyley.github.io/bfg-repo-cleaner/#download) and put it somewhere
3. `cd {git repo}`
4. `java -jar {bfg_dir}/bfg-1.12.8.jar --convert-to-git-lfs '*.dll' --no-blob-protection`
5. `java -jar {bfg_dir}/bfg-1.12.8.jar --convert-to-git-lfs '*.exe' --no-blob-protection`
6. And so on, one pattern at a time. This is the current recommendation.
7. `.gitattributes` are not generated through out history... So the best bet is to start tracking on the newest version only. This WILL go bad if you go to a previous version of history, branch off, commit, as LFS track files will not know to track at that point. I guess this would have to be fixed via ` git-filter-branch`
8. `git lfs track '*.dll'` '*.exe'`
10. For all the patterns
11. `git add .gitattributes`
12. `git commit -m "Added .gitattributes for lfs tracking"`
13. Final step, `git push origin --all -f` and `git push origin --tags -f`. The `-f` is the important point of no return. This pushes a DIFFERENT repo now. Anyone else out there who has a clone, will have the broken clone, and will need to re-clone the new better lfs version.
14. Don't be alarmed that `git status` shows all your files as different. This is a common git issue where the index is confused. It can be fixed by.... Well, not sure the best way. Throw away the clone, and clone again is one way, checkout initial commit and then go back is another (this only works if none of the affected files exist in the initial commit). Both of these are BAD solutions. The best way is _____?
