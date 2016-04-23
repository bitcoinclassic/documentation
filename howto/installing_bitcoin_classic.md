#How to install Bitcoin Classic

This document contains some instructions on how to install Bitcoin Classic.

##Installing from pre-built binary packages

The Classic project provides some pre-built packages for popular operating systems.
These can be used to install quickly and easily, making use of the package management features present in your OS.

###Installing on Ubuntu

The Classic project maintains a package repository at https://launchpad.net/~bitcoinclassic/+archive/ubuntu/bitcoinclassic/

Currently, there are only packages for the Long Term Support version 14.4 (trusty).

####Installing on Ubuntu 14.4 LTS

#####Removing other clients / repos
if you have other Bitcoin repositories or clients installed, you should remove these using the package management features. Otherwise, you may skip this section.

You can check whether this is the case by running

    dpkg -l | grep bitcoin

If the output lists a `bitcoin-qt`or `bitcoind`package, you have some other Bitcoin software installed.

You can also check for installed Bitcoin repositories using

    apt-cache policy | grep bitcoin

To remove other Bitcoin software, ensure that it is not running (shut it down cleanly as necessary).

Then remove its package by running

    sudo apt-get remove bitcoin-qt

or

    sudo apt-get remove bitcoind

depending on whether the graphical client (bitcoin-qt) or the headless client (bitcoind) or both were installed. Continue only if the removal of the package(s) was successful.

As a next step, remove other Bitcoin software repositories, as their packages may conflict with those of Classic.

For example, if you have the Bitcoin Core PPA installed previously, you can remove it as follows:

    sudo apt-add-repository --remove ppa:bitcoin/bitcoin

Running the `apt-cache policy | grep bitcoin` command should now produce an empty output.

You are now ready to install the Classic PPA and software packages.

#####Installing the Classic repository information (PPA)

The following command will install the repository information for Bitcoin Classic:

    sudo apt-add-repository ppa:bitcoinclassic/bitcoinclassic

If that is successful, you should update the available package information using

    sudo apt-get update

#####Installing the Classic client software

There are two software packages which you can install:

- bitcoin-qt, the graphical client
- bitcoind, the headless client (usually run as a daemon)

You can install either or both of these with 

    sudo apt-get install bitcoin-qt
    sudo apt-get install bitcoind

They can also be installed together (although only one of them can be run at a time).

After installation, you should have the respective binary package installed in `/usr/bin/`

###Installing on generic Linux

Prebuilt binaries are also provided in compressed tar archives (.tgz) for 32- and 64-bit Linux systems.

Attempting to run the 64-bit binaries on a 32-bit machine will fail. If in doubt, you can check your Linux using

    uname -m

If the output is `x86_64`you should obtain the 64-bit version, otherwise the 32-bit version.

Without going into full details, the installation steps are as follows:

1. Download the appropriate tar archive. 
The "Download" link at https://bitcoinclassic.com will take you to the latest release, where you should find .tar.gz files for 32- and 64 bit Linux systems.

2. Unpack it in a place of your choice.
The software can be run from any ordinary user's folder. For example:

        cd /path/where/you/want/to/unpack
        tar xf bitcoin-0.x.y-linux64.tar.gz

    In the example above, `x` and `y` indicate software versions.
    This will create a versioned subfolder, e.g. `bitcoin-0.x`, containing the Classic software.

3. Run the executable from its installation location (if necessary adapting your PATH setting)

        /path/where/you/unpacked/bitcoin-0.x.y/bin/bitcoin-qt

The archives contain both the graphical and headless clients. The executables are contained in the `bin` subfolder:

    bitcoin-0.x.y/bin/bitcoind
    bitcoin-0.x.y/bin/bitcoin-qt

There are also other binaries such as the command line RPC client, `bitcoin-cli`.

##Installing on Windows

Obtain the appropriate installer by following the "Download" link at https://bitcoinclassic.com which will take you to the latest release.  There you will find .exe files for 32- and 64 bit Windows systems.
They are named e.g.

    bitcoin-0.x.y-win32-setup.exe
    bitcoin-0.x.y-win64-setup.exe

where x and y are version numbers.

Run the setup program to complete the installation steps.

##Installing from sources

Instructions on how to build Bitcoin for your platform from source code are included in the source code package. Download the source code by following the "Download" link at https://bitcoinclassic.com and selecting one of the source packages.

After extracting it on your system, consult the `doc/build-<platform>.md` files for more information on how to build it.
