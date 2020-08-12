# RescueOS build HOWTO

These instructions are for folks who want to modify RescueOS and make their own
images.  If you just want to use RescueOS on your N900, you'll want to get one
of the prebuilt images and look at the [README](README.md) for usage info.

RescueOS images are modified Buildroot images.  This source distribution is just
a collection of configuration files and patches that configure and customize
Buildroot builds.

You'll need a GNU/Linux system to host the build.  You'll need `git` and `quilt`
to checkout the RescueOS source and apply the Buildroot patches, respectively.
Once that's done, you'll also need to satisfy Buildroot's prerequisites; we'll
address that later.

These files are meant to be used with version 2013.02 of Buildroot.  (Ports to
newer versions are welcome.)

Start by unpacking the RescueOS and Buildroot sources into separate directories.
We recommend putting the two directories next to each other under a common
parent directory:

```shell
mkdir rescueos-build
cd rescueos-build
```

Use `git` to checkout the RescueOS source.  It will go into a subdirectory named
`N900_RescueOS`.

```shell
git clone <RescueOS repository URL>
```

Fetch the Buildroot source tarball and checksums files.

```shell
wget http://www.buildroot.org/downloads/buildroot-2013.02.tar.bz2
wget http://www.buildroot.org/downloads/buildroot-2013.02.tar.bz2.sign
```

The checksums file (ending with `.sign`) is a text file with an inline PGP
signature.  Verify the signature and check the tarball with the checksum.

```shell
gpg --recv-keys B025BA8B59C36319
gpg --output buildroot-2013.02.tar.bz2.sums buildroot-2013.02.tar.bz2.sign
grep '^SHA1: ' buildroot-2013.02.tar.bz2.sums | sha1sum --check
```

Unpack Buildroot.  It will end up in a subdirectory named `buildroot-2013.02`.

```shell
tar --extract --bzip --file buildroot-2013.02.tar.bz2
```

Buildroot's prerequisites are documented in the "System requirements" section of
the Buildroot manual (`buildroot-2013.02/docs/manual/manual.html`).  Install
them if necessary.

Now apply our Buildroot patches with `quilt`.

```shell
cd buildroot-2013.02
QUILT_PATCHES=../N900_RescueOS/buildroot-patches quilt push -a
```

Point buildroot to the RescueOS configuration:

```shell
make defconfig BR2_DEFCONFIG=../N900_RescueOS/buildrootconfig
```

Invoke buildroot to build RescueOS by running `make`.

Buildroot will download and build a whole bunch of packages, starting with the
cross-compiling toolchain.  When it's done, the kernel and cramfs images will be
left in `buildroot-2013.02/output/images`.
