Git LFS v2.0.0 includes an early release of File Locking. File Locking lets
developers lock files they are updating to prevent other users from updating
them at the same time. Concurrent edits in Git repositories will lead to merge
conflicts, which are very difficult to resolve in large binary files.

## Tracking Lockable files

Git has built-in tools for resolving merge conflicts in text files (such as
source code, documentation, etc). The first step to using File Locking is to
define what file types need the extra overhead. The `git lfs track` command
includes a `--lockable` flag. The following command will store JPG files in LFS,
and mark them as lockable:

```sh
$ git lfs track "*.jpg" --lockable
```

This adds the following line to your `.gitattributes` file:

```sh
*.jpg filter=lfs diff=lfs merge=lfs -text lockable
```

If you'd like to register a file type as lockable, without using LFS, you can
edit the `.gitattributes` file directly:

```sh
*.yml lockable
```

Once file patterns in `.gitattributes` are lockable, Git LFS will make them
readonly on the local file system automatically. This prevents users from
accidentally editing a file without locking it first.

## Managing Locked Files

When you are ready to edit files, run the `lock` command:

```sh
$ git lfs lock images/foo.jpg
Locked images/foo.jpg
```

This registers the file as locked in your name on the server. You can view this
with the `locks` command.

```sh
$ git lfs locks
images/bar.jpg  jane   ID:123
images/foo.jpg  alice  ID:456
```

The file will also be ready for you to edit and push locally. If at any time
you decide you don't need the lock, you can remove it by passing the path or ID
to the `unlock` command.

```sh
$ git lfs unlock images/foo.jpg
$ git lfs unlock --id=456
```

You can also unlock someone else's file with the `--force` flag:

```sh
$ git lfs unlock images/foo.jpg --force
```

Note: Different LFS server implementations may have different permissions. Some
may require admin privileges to unlock someone else's lock, for example.

## Pushing Your Changes

Git LFS will verify that you're not modifying a file locked by another user when
pushing. Since File Locking is an early release, and few LFS servers implement
the API, Git LFS won't halt your push if it cannot verify locked files. You'll
see a message like this:

```sh
$ git lfs push origin master --all
Remote "origin" does not support the LFS locking API. Consider disabling it with:
  $ git config 'lfs.http://git-server.com/user/test.locksverify' false
Git LFS: (0 of 0 files, 7 skipped) 0 B / 0 B, 879.11 KB skipped
```

```sh
$ git lfs push origin master --all
Locking support detected on remote "origin". Consider enabling it with:
  $ git config 'lfs.http://git-server.com/user/repo.locksverify' true
Git LFS: (0 of 0 files, 7 skipped) 0 B / 0 B, 879.11 KB skipped
```