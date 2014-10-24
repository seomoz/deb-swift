# deb-swift

[![Build Status](https://travis-ci.org/seomoz/deb-swift.svg?branch=master)](https://travis-ci.org/seomoz/deb-swift)

`deb-swift` is a simple utility to make creating and managing APT repositories on
OpenStack Swift.

This shamelessly steals from the deb-s3 project (https://github.com/krobertson/deb-s3), and is separate only because I was unable to get the aws-sdk gem (on which deb-s3 depends) to work cleanly with a non-S3 object store (swift).

# Features

* Downloads the existing package manifest and parses it.
* Updates it with the new package, replacing the existing entry if already
  there or adding a new one if not.
* Uploads the package itself, the Packages manifest, and the Packages.gz
  manifest. It will skip the uploading if the package is already there.
* Updates the Release file with the new hashes and file sizes.

## Getting Started

You can simply install it from rubygems:

```console
$ gem install deb-swift
```

Or to run the code directly, just check out the repo and run Bundler to ensure
all dependencies are installed:

```console
$ git clone https://github.com/seomoz/deb-swift.git
$ cd deb-swift
$ bundle install
```

Now to upload a package, simply use:

```console
$ deb-swift upload --bucket my-bucket my-deb-package-1.0.0_amd64.deb
>> Examining package file my-deb-package-1.0.0_amd64.deb
>> Retrieving existing package manifest
>> Uploading package and new manifests to Swift
   -- Transferring pool/m/my/my-deb-package-1.0.0_amd64.deb
   -- Transferring dists/stable/main/binary-amd64/Packages
   -- Transferring dists/stable/main/binary-amd64/Packages.gz
   -- Transferring dists/stable/Release
>> Update complete.
```

```
Usage:
  deb-swift upload FILES

Options:
  -a, [--arch=ARCH]                     # The architecture of the package in the APT repository.
      [--sign=SIGN]                     # Sign the Release file. Use --sign with your key ID to use a specific key.
  -p, [--preserve-versions]             # Whether to preserve other versions of a package in the repository when uploading one.
  -b, [--bucket=BUCKET]                 # The name of the Swift bucket to upload to.
  -c, [--codename=CODENAME]             # The codename of the APT repository.
                                        # Default: stable
  -m, [--component=COMPONENT]           # The component of the APT repository.
                                        # Default: main
      [--access-key-id=ACCESS_KEY]      # The access key for connecting to Swift.
      [--secret-access-key=SECRET_KEY]  # The secret key for connecting to Swift.
  -v, [--visibility=VISIBILITY]         # The access policy for the uploaded files. Can be public, private, or authenticated.
                                        # Default: public

Uploads the given files to a Swift bucket as an APT repository.
```

You can also delete packages from the APT repository. Please keep in mind that
this does NOT delete the .deb file itself, it only removes it from the list of
packages in the specified component, codename and architecture.

Now to delete the package:
```console
$ deb-swift delete --arch amd64 --bucket my-bucket --versions 1.0.0 my-deb-package
>> Retrieving existing manifests
   -- Deleting my-deb-package version 1.0.0
>> Uploading new manifests to Swift
   -- Transferring dists/stable/main/binary-amd64/Packages
   -- Transferring dists/stable/main/binary-amd64/Packages.gz
   -- Transferring dists/stable/Release
>> Update complete.

````

You can also verify an existing APT repository on Swift using the `verify` command:

```console
deb-swift verify -b my-bucket
>> Retrieving existing manifests
>> Checking for missing packages in: stable/main i386
>> Checking for missing packages in: stable/main amd64
>> Checking for missing packages in: stable/main all
```

```
Usage:
  deb-swift verify

Options:
  -f, [--fix-manifests]                 # Whether to fix problems in manifests when verifying.
      [--sign=SIGN]                     # Sign the Release file. Use --sign with your key ID to use a specific key.
  -b, [--bucket=BUCKET]                 # The name of the Swift bucket to upload to.
  -c, [--codename=CODENAME]             # The codename of the APT repository.
                                        # Default: stable
  -m, [--component=COMPONENT]           # The component of the APT repository.
                                        # Default: main
      [--access-key-id=ACCESS_KEY]      # The access key for connecting to Swift.
      [--secret-access-key=SECRET_KEY]  # The secret key for connecting to Swift.
  -v, [--visibility=VISIBILITY]         # The access policy for the uploaded files. Can be public, private, or authenticated.
                                        # Default: public

Verifies that the files in the package manifests exist
```

## TODO

This is still experimental. There are several things to be done.
