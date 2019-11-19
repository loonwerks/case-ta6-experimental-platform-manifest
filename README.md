# DARPA CASE Program Experimental Platform

Welcome! The repository hosts the 'Git Repo' specification for the manifest
aggregating the CASE experimental platform software base.  This manifest
contains references to pull dockerfiles for building a common build
environment, the OpenUxAS appliction and the LmcpGen tool used for generating
implementation code for the UxAS message specifications.


## Requirements:

 * Git Repo (See below for installation instructions.)
 * Docker (See here for instructions: https://get.docker.com or https://docs.docker.com/engine/installation.)
   Further instructions are available in the dockerfiles/README.md.
 * Make

It is recommended you add yourself to the `docker` group, so you can run
Docker commands without using sudo.

## Installing Git Repo

Repo is a tool that makes makes the process of manageing aggregated
collections of Git repositories easier and more consistent. For more
information about Repo, see the
[Repo Command Reference](https://source.android.com/setup/develop/repo).

To install Repo:

Make sure you have a bin/ directory in your home directory and that it is included in your path:

~~~
mkdir ~/bin
PATH=~/bin:$PATH
~~~

Download the Repo tool and ensure that it is executable:

~~~
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
~~~

## Quick start (for phase-2 platform assessment #1 ODROID XU4 target):

The following instructions should fetch and build the baseline experimental platform (assuming a Bash shell):

~~~
mkdir case-ta6-experimental-platform-manifest
cd case-ta6-experimental-platform-manifest
repo init -u https://github.com/loonwerks/case-ta6-experimental-platform-manifest.git
repo sync
cd dockerfiles
HOST_DIR=`pwd`/.. DOCKER_TAG=p2pa1 UXAS_BRANCH=develop-case-ta6-ph2-example01 make uxas_build
./build-sd-card.sh <dev>
~~~

These instructions apply Docker containers to fetch necessary prerequisites,
compilation tools, Uboot and Linux images, etc. to construct a bootable SD card
for the ODROID XU4 containing an OpenUxAS executable and configuration files.

As a prerequisite to applying these instructions, the Docker service must be
installed on your host machine.  If this has not already been done, please see
instructions at the CASE TA6 experimental platform dockerfiles repository in
the README.md file.

The first four steps apply the Google `repo' tool to make checkouts of CASE TA6
experimental platform related repositories.  The sixth step applies a Makefile
to call `docker build` to construct Docker images containing all of the necessary
build infrastructure and to build an OpenUxAS executable for the ARMv7 target.

The final step automates zeroizing an SD memory card and writing preboot code,
bootloaders, a Linux installation and the build OpenUxAS software.  The
build-sd-card.sh script takes a single parameter <dev> giving the device name
of the raw SD card.

The SD card may then be inserted in the ODROID XU4 and the hardware powered on.
Note that on the first boot, several items including crypto keys are initialized
and then the Linux kernel will halt.  On the second and subsequent boots, Linux
will reach a login prompt were the username `uxas` and password `uxas` may be
given to reach a shell prompt.

The phase 2 platform assessment 1 configurations should be found in the
examples/CASE-TA6-Challenge-Problems/ph2_01_WaterwaySearch directory.
The cfg_WaterwaySearch_GS.xml file will likely need to be edited to name the
network interface to be used (typically `eth0`) and the IP address of the host
on which the AMASE visualization is to be run.  The ground station may then be
invoked with the `./runUxAS_WaterwaySearch_GS.sh` script.

## Obtaining a build environment

To get a running build environment for OpenUxAS targeting the ARMv7, run:

~~~
mkdir case-ta6-experimental-platform-manifest
cd case-ta6-experimental-platform-manifest
repo init -u https://github.com/loonwerks/case-ta6-experimental-platform-manifest.git
repo sync 
cd dockerfiles
HOST_DIR=`pwd`/.. make user
~~~

This will build and run Docker images resulting in a shell containing all of
the necessary tools and dependencies to cross build the OpenUxAS application
for the target ODROID XU4 platform.  The build environment also contains the
boot loader and Linux operating software used to construct a bootable
MicroSD card.

Then in the build environment shell follow build from the top-level ANT
script:

~~~
ant compile
~~~

This will follow build the LMCP message compiler, compile the LMCP messages
to source code, prepare and configure the OpenUxAS build tree, and compile
the OpenUxAS application from source to a binary for the ODROID XU4 target.


## Building OpenUxAS

The top-level ANT build script followsthe build instructions for OpenUxAS
as follows:

~~~
cd LmcpGen
ant jar
cd ../OpenUxAS
./RunLmcpGen.sh
./prepare
meson build-armhf --cross-file=arm-linux-gnueabihf-cross-file.txt --buildtype=release
ninja -C build-armhf all
~~~


## Adding Dependencies

The images and dockerfiles for the experimental platform collect only the
dependencies to build OpenUxAS and pass the compilation test in the
case-ta6-uxas-build.dockerfile.  The `extras.dockerfile` acts as a shim
between the DockerHub images and the `user.dockerfile`. 

Adding dependencies into the `extras.dockerfile` will build them the next time
you run `make user`, and then be cached from then on.


## All (useful) Make Commands

Specification of the following targets to the `make` command are possibly
useful to users of the experimental plaform build environment.

### Starting a container from Docker Registry

    user                   # Alias for user_uxas_tools
    user_odroid_xu4_tools  # Start a container with ODROID XU4 cross build tools
    user_odroid_xu4_build  # Start a container with ODROID XU4 cross build tools + Bootloader/Linux dependencies
    user_uxas_tools        # Start a container with ODROID XU4 cross build tools + Bootloader/Linux dependencies + UxAS cross build tools and dependencies

### Getting the images from Docker Registry

    pull_odroid_xu4_tools_image    # Pull the ODROID XU4 tools image from Docker registry
    pull_odroid_xu4_build_image    # Pull the ODROID XU4 cross build tools + Bootloader/Linux dependencies image from Docker registry
    pull_uxas_tools_image          # Pull the ODROID XU4 cross build tools + Bootloader/Linux dependencies + UxAS cross build tools and dependencies image from Docker registry
    pull_images_from_dockerhub     # Pull all the above images from Docker registry

### Building the Local Dockerfiles

    base_tools
    odroid_xu4_tools
    odroid_xu4_build
    uxas_tools
    all

    rebuild_base_tools
    rebuild_odroid_xu4_tools
    rebuild_odroid_xu4_build
    rebuild_uxas_tools
    rebuild_all

### Testing the Local Dockerfiles

    test_uxas

    retest_uxas

    run_tests
    rerun_tests


## Fusing the Build onto a MicroSD Card

Ideally, this is as simple as running (assuming you have a bash shell):

~~~
./build-sd-card.sh <dev>
~~~

where `<dev>` is the device file for the raw MicroSD card (e.g. /dev/sdc).
This script erases, partitions and formats the MicroSD card, gathers the
various elements of software including the bootloader, device tree,
boot filesystem and root filesystem, UxAS application and example
configuration files and fuses these onto the MicroSD card.

In future, we hope to have a script working in a platform independent
manner under the Docker-based build system that constructs a whole-disk
image file.  This file would then support using fusing tools for for
various host development platforms.


## To Do

1. Construct a script to build a whole-disk image for the MicroSD card rather
   than relying on the complexities in the current build-sd-card.sh script.
   This will allow use of a variety host-platform specific image writing tools
   rather than depending on Bash and DD.

