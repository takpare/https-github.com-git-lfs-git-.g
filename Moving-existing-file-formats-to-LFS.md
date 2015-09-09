Moving to LFS does not have to be hard! The Windows batch script below can be used to migrate one file format at a time. Simply create a batch file with the source below inside the root of your repository, and call it as follows: `MyBatchFile.bat exe`, in order to move all ".exe" files to LFS!

    SET file_extension=%~1

    :: Remove all the desired files from VCS, but keep them locally
    git rm --cached "*.%file_extension%"
    git commit -m "Removed %file_extension% format from the repository"

    :: Tell LFS to track this format, has to be done before the files are added
    git lfs track "*.%file_extension%"

    :: Make sure to add the changes made by LFS above
    git add .gitattributes

    :: Add to LFS!
    git add "*.%file_extension%"
    git commit -m "Added %file_extension% format to LFS"​
    
Note that the example above does *not* remove usage of the file format from history, see [Remove Sensitive Data](https://help.github.com/articles/remove-sensitive-data/) for more information on that.