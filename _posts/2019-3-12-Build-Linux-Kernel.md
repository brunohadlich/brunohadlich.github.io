---
layout: post
title: Building and installing your own Linux kernel on Debian based distros!
---

First thing to do is assure that the necessary packages are installed with:

sudo apt-get update

sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc bison flex gcc make exuberant-ctags gawk gettext linux-tools-common linux-tools-generic linux-cloud-tools-generic

After that you will need to clone Linux repository, this can take some time depending on your internet connection:

git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

Access the downloaded repository:

cd linux-stable

Create a new branch called stable:

git checkout -b stable

List the available tags with:

git tag

Select the tag that corresponds to the version you wish to build replacing v5.0 with the one you want:

git checkout tags/v5.0

Copy your own Kernel Configuration file to ensure your new kernel will keep the drivers your computer already uses.

sudo cp /boot/config-`uname -r`* .config

Run menuconfig, do not change anything unless you know what you are doing, save it and exit.

make menuconfig

Run the following commands to build, install modules and then install the kernel itself. The example below is using 4 threads, in case you computer has a different number of threads you can change this value.

sudo make -j 4
sudo make modules_install -j 4
sudo make install -j 4
