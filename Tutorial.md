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

## Looking at the point

`git show HEAD:{filename}`

## Adding git-lfs to a pre-exiting repo  ##

TODO