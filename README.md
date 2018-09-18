# DXL Datasets

This is a repository using [git-annex](https://git-annex.branchable.com/) to
store datasets under use by the [DXL](https://data-exp-lab.github.io/) at the
University of Illinois Urbana-Champaign.

This approach -- using git annex -- allows very deliberate motion of data,
ensuring that we can track multiple datasets without requiring that they
simultaneously be "checked out" in a given repository.  You can dump data here
and there and only have it take up space on your disk when you need it.

At present the primary backend for storing datasets is using
[box.illinois.edu](https://box.illinois.edu/) using a
["specialremote"](https://git-annex.branchable.com/tips/using_box.com_as_a_special_remote/).
Files are stored using non-human readable path names, so when you access your
box you will not be able to do very much with them; they will only really be
accessible through git.

From time to time we will mirror these files outside of Box, but in general
they'll be stored there.  As often as possible, public (read-only) HTTP URLs
will be added as additional sources in git annex, to ensure this repository is
useful to those without access to UIUC Box.

## Getting Access

When you join the lab, you will be granted read/write access to the dxl box
folder.  It's called `dxl-git-annex`.

Once you have obtained the shared folder and it is in your top-level Box
folder, you will need to set up a special password for Box that allows access
outside of the UIUC Single Sign On.  You can do this by going to "Account
Settings" and finding "Authentication."

**Be sure to disable notifications from Illinois's Box for the `dxl-git-annex`
folder!** Otherwise you'll get email notifications all the time.  And they're
annoying and unnecessary.

## Getting Set Up

Before you can use this repository to its full advantage, you need to install
`git annex`.  On OSX this can be done via homebrew or manually, and on linux it
tends to come with your package manager.

To get set up, you need to initially clone this repository, initialize the
annex, and enable the box specialremote.  In this set of commands, replace the
value of `WEBDAV_USERNAME` and `WEBDAV_PASSWORD` with your netid and whatever
you set your WEBDAV password to be.

```
git clone git@github.com:data-exp-lab/datasets/
cd datasets
git annex init
WEBDAV_USERNAME=yournetid@illinois.edu WEBDAV_PASSWORD="whatever" git annex enableremote box
```

At this point, you will have your annex enabled.

When there are new changes you want to get, you can use the command

```
git annex sync
```

This will synchronize the state of your repository with the upstream one, making locations of data obvious.

## Getting a Dataset

To obtain a file or dataset, you can execute this command:

```
git annex get FILETOGET
```

in this case, replace `FILETOGET` with the file you want to get; this will work
recursively if given a directory name.

## Freeing Up Space

If you use the command `git annex drop` on a file managed by git annex, it will
remove the file from your machine, but will *not* remove it from the remote.

This is really cool.  It means that when you're sick of having a file around
you can remove it locally, but then re-obtain it when you need it.

## Adding Datasets

The process by which you add a dataset is reasonably simple, and familiar to
git users.  The rules are:

 * Each dataset gets its own directory, even if it is a single file
 * Each directroy needs a README with a bit of info -- what the data *is*,
   where it originated from, and what restrictions there are on its use
   (including a citation if one is available).  This helps us track what it is
   we're looking at.
 * Any scripts you have that are helpful would be appreciated.

**README files and scripts do not have to be managed by annex**.  They can just
be managed by regular git.

Once you've created that, you put the data in your directory and use the `git
annex add` command.  This will add an entry to the annex metadata and it will
change the file in the current directory from an actual file into a symbolic
link.

The most important step is then moving it to our remote -- for instance, box.
This can be done with the command:

```
git annex copy FILENAME --to box
```

So for a worked example, if I have files `mydata1.hdf5` and `mydata2.hdf5` that
I wanted to add, I would:

```
git annex sync
mkdir mydata
cd mydata
cp ~/mydata-readme ./README
cp ~/mydata1.hdf5 .
cp ~/mydata2.hdf5 .
git add README
git annex add mydata1.hdf5 mydata2.hdf5
git annex copy mydata1.hdf5 mydata2.hdf5 --to box
git commit
git annex sync
```

This will add the data, copy it to box, and then sync your repository with the
hosted one.

You can then, if you want, `git annex drop mydata1.hdf5 mydata2.hdf5` to free
up space locally, safe and secury in the knowledge that you will be able to get
them again.
