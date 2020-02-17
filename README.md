Create differential tarballs
============================

Note: The https://github.com/regro/conda-mirror/pull/9 adds this tool
      to conda-mirror.

This tools allows you to create differential tarballs of a (usually
mirrored) conda repository.  The resulting tarball can be used to update
a copy of the mirror on a remote (air-gapped) system, without having to
copy the entire conda repository.  The workflow is a follows:

  1. we assume that the remote and local repository are in sync
  2. create a `reference.json` file of the local repository
  3. update the local repository using `conda-mirror` or some other tools
  4. create the "differential" tarball
  5. move the differential tarball to the remote machine, and unpack it
  6. now that the remote repository is up-to-date, we should create a new
     `reference.json` on the local machine.  That is, step 2

Notes:
------

The file `reference.json` is a collection of all `repodata.json`
files (`linux-64`, `win-32`, `noarch`, etc.) of the local repository.
It is created in order to compare a future state of the repository to the
state of the repository when `reference.json` it was created.

The differential tarball contains files which either have been updated (such
as `repodata.json`) or new files (new conda packages).  It is meant to be
unpacked on top of the existing mirror on the remote machine by:

    cd <repository>
    tar xf update.tar
    # or y using tar's -C option from any directory
    tar xf update.tar -C <repository>


Example:
--------

In this example we assume that a conda mirror is located in `./repo`.
Create `reference.json`:

    python mk_diff_tar.py --reference ./repo

Show the files in respect to the latest reference point file (which would be
included in the differential tarball).  Since we just created the reference
file, we don't expect any output:

    python mk_diff_tar.py --show ./repo

Now, we can update the mirror:

    conda-mirror --upstream-channel conda-forge --target-directory ./repo ...

Create the actual differential tarball:

    $ python mk_diff_tar.py --create ./repo
    Wrote: update.tar
    $ tar tf update.tar
    noarch/repodata.json
    noarch/repodata.json.bz2
    noarch/ablog-0.9.2-py_0.tar.bz2
    noarch/aws-amicleaner-0.2.2-py_0.tar.bz2
    ...
