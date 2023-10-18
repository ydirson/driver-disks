# Driver Disk generation for XCP-ng

Driver Disks are a specific kind of Supplemental Packs for XCP-ng and
XenServer.  While XenServer supports intalling arbitrary packages as
Supplemental Packs and XCP-ng 8.2 did not prevent from attempting to
install them, starting with 8.3 XCP-ng the installer will only allow
to install Driver Disks, targeting the very specific situation where
XCP-ng could not be installed on a given hardware using the
installation ISO alone (waiting for a new installation ISO with
updated hardware support).

An XCP-ng Driver Disk is meant to install a single driver RPM package.

This repository contains scripts to generate driver disks, as well as
configuration files for producing Driver Disks.

* `create-driver-disk`: the main script, taking a configuration
  file as parameter, and creating a Driver Disk ISO.
* `generate-test-key`: a tool to generate a Driver Disk signing key
  for testing purposes.  Can also be used as inspiration to generate
  your own official signing key.
* `*.cfg`: configuration files, each describing a driver disk.

You are encouraged to fork this repository to store your own Driver
Disk definitions.

## Running the tool

`create-driver-disk` requires the XenServer `build-update` tool, to be
found in the `update-package` package.  The easiest way is to run
inside the [XCP-ng build-environment docker
container](https://github.com/xcp-ng/xcp-ng-build-env) (which you
already have to setup to build a driver RPM), with additional package
`update-package` installed.

To create the Driver Disk for the `mpi3mr` Broadcom RAID driver, not
present in the original XCP-ng 8.2.1 installation ISO:

* get a copy of the [driver
  RPM](https://updates.xcp-ng.org/8/8.2/updates/x86_64/Packages/mpi3mr-module-8.6.1.0.0-1.xcpng8.2.x86_64.rpm)
  in the current directory
* <details>
    <summary>generate a test gpg key and store the public key in the expected place</summary>

    ```
    [builder@43e15989a9bf driver-disks]$ ./generate-test-key RPM-GPG-KEY-XS-DDK-Test
    Passphrase: 
    /home/builder/.gnupg/pubring.gpg
    --------------------------------
    pub   2048R/E8BB51DB 2023-10-18 [expires: 2024-04-15]
    uid                  Example Updates (update) <example@example.com>
    
    gpg: Generating a test update key
    gpg: writing self signature
    gpg: RSA/SHA1 signature from: "A16266FA [?]"
    gpg: writing public key to `/home/builder/.gnupg/pubring.gpg'
    gpg: writing secret key to `/home/builder/.gnupg/secring.gpg'
    gpg: using PGP trust model
    gpg: key A16266FA marked as ultimately trusted
    gpg: done
    ```
  </details>
* <details>
    <summary>run the script (inside the docker container)</summary>

    ```
    [builder@43e15989a9bf driver-disks]$ ./create-driver-disk intel-e1000e-alt-8.2.cfg
    intel-e1000e-alt-3.8.7-1.xcpng8.2.x86_64.rpm: OK
    Enter passphrase for Example Updates (update) <example@example.com>: 
    error: Macro %build_number has empty body
    error: Macro %build_number has empty body
    + umask 022
    + cd /tmp/build-update-ncjV13/BUILD
    + /usr/bin/rm -rf /tmp/build-update-ncjV13/BUILDROOT/update-intel-e1000e-alt-3.8.7.1.xcpng8.2.pack.0.1-1.x86_64
    + exit 0
    /tmp/build-update-ncjV13/root/Packages/intel-e1000e-alt-3.8.7-1.xcpng8.2.x86_64.rpm:
    /tmp/build-update-ncjV13/root/Packages/update-intel-e1000e-alt-3.8.7.1.xcpng8.2.pack.0.1-1.noarch.rpm:
    Spawning worker 0 with 1 pkgs
    Spawning worker 1 with 1 pkgs
    Spawning worker 2 with 0 pkgs
    Spawning worker 3 with 0 pkgs
    Workers Finished
    Saving Primary metadata
    Saving file lists metadata
    Saving other metadata
    Generating sqlite DBs
    Sqlite DBs complete
    Total translation table size: 0
    Total rockridge attributes bytes: 2468
    Total directory bytes: 6144
    Path table size(bytes): 42
    Max brk space used 0
    261 extents written (0 MB)
    ```
  </details>

Please note the `error: Macro %build_number has empty body` is an
upstream "normal error", and can be safely ignored.

## Writing a configuration file

Configuration files are a set of key/value pairs, in the form of shell
statements.  Make sure to use no space on either side of the `=` sign,
and to quote the value, especially if it contains any space characters.

Using `TEMPLATE.cfg` as base instead of a fully-working one will avoid
avoidable mistakes like not selecting a unique UUID.

A valid config file must contain the following keys:

* `PACK_DESC`: textual description for the pack, which will be shown
  on the hosts where the pack is installed
* `PACK_UUID`: a UUID to describe the pack and link together its
  successive versions; use `uuidgen` to create a one
* `PACK_BUILD`: a "build number"/revision for this pack, used together
  with the XCP-ng version to build a version number for the Driver Disk
* `RPM_FILE`: (path to) driver RPM file to install; can be a path to
  another location, or the RPM can be copied locally
* `RPM_SHA256`: cryptographic hash for the RPM file, for verification
  purposes a build time
* `GPG_PUBKEY_FILE`: (path to) file containing the public key
  indicating which private key to use for signing the Driver Disk
