cowpatch
========

This is a "copy-on-write" patch application tool. Think "overlayfs for
patching big source trees, but doesn't need Linux or root". Its
intended audience is people who need to script building of large
software with patches applied, especially for testing different
iterations or combinations of patches. cowpatch allows you to keep a
single "pristine" tree of the upstream source, and do all the work on
ultrathin patched clones.

Please be aware that the "copy-on-write" property applies only at
patch time. If the build process modifies its own tree in any way, it
can and will clobber the pristine tree. Use of cowpatch is recommended
only with software that's capable of "out-of-tree" builds.

Warning
-------

**THIS IS EXPERIMENTAL SOFTWARE**. It does not perform any fancy
checks like `patch` would do before acting on a file. Malformed or
malicious diffs, or even just bugs, may cause it to perform
destructive acts outsie of the tree you intend to patch.

Usage
-----

cowpatch should be run with the same options you'd pass to patch
(especially `-pN` as appropriate) in a directory initially populated
with nothing but symlinks to entries in the pristine tree.

In addition, cowpatch-specific options `-I` and `-S` are available to
initialize or manually split files in a copy-on-write tree.

Example
-------

Do something like the following to make a new cow clone and apply a
patch:

    mkdir gcc-patched
    cd gcc-patched
    cowpatch.sh -I ../gcc-orig
    cowpatch.sh -p1 < ../gcc-changes.diff

Here, the `cowpatch.sh -I ../gcc-orig` command behaves like `ln -s
../gcc-orig/* .` except that it correctly handles hidden files (ones
whose names start with `.`).

Note that it's important to start with a directory full of symlinks,
not just a symlink to the base directory. The following will **NOT**
work:

    ln -s gcc-orig gcc-patched
    cd gcc-patched
    ...

because the cd simply changes into the symlink target directory.

To manually split a particular file (e.g. for editing) without a patch
that modifies it, you can use the `-S` option. For example:

    cowpatch.sh -S configure

Notes
-----

You can apply more than one patch this way. Application of each
additional patch will "cow" split only the directories/files it
applies to.
