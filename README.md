**Custom kernel build with the touchpad patch applied for steberg**

**Prerequisites to build from source**

You will need to install the following packages:

    sudo apt-get install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex libelf-dev


Fetch sources for current stable build:

    mkdir -p ~/linux && cd ~/linux
    wget -c -v -nc https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.18.9.tar.xz

Then set up build dependencies (may fail if no source entries are enabled):

 

    apt-get build-dep linux-image-$(uname -r)


**Kernel Build and Installation:**

Create a workspace to hold the necessary patchwork:

    mkdir -p ~/linux/patchwork

Fetch the patches:

    cd ~/linux/patchwork

Fetch with wget:

    wget -c -v -nc "https://bugzilla.kernel.org/attachment.cgi?id=278573&action=diff&collapsed=&headers=1&format=raw" -O touchpad-patch.patch

Then navigate to the kernel source and extract:

    cd ~/linux
    tar -xvf linux-4.18.9.tar.xz

Navigate to the source tree and apply the patch:

    cd linux-4.18.9

1. Dry run to confirm any errors:

    patch -p1 --dry-run < ../patches/touchpad-patch.patch

2. Success. Now patch:

    patch -p1 < ../patches/touchpad-patch.patch


Now, proceed as shown below:

Copy the kernel config file from your existing system to the kernel tree:

    cp /boot/config-`uname -r` .config

Bring the config file up to date:

    yes '' | make oldconfig


**Optional:** If you need to make any kernel config changes, do the following and save your changes when prompted:

    make menuconfig


Build the linux-image and linux-header .deb files using a thread per core + 1. This process takes a lot of time:

    time make -j `getconf _NPROCESSORS_ONLN` deb-pkg  VERBOSE=1 

**Optional steps (not needed for your case, but documented for reference):**

If you need the binary artifacts too (for facilities such as the cloud-tools package):

    fakeroot debian/rules binary

To test normal package building for all flavors:

    fakeroot debian/rules binary-arch

This will build every kernel flavour.

When done with the build, you'll find the artifacts (debian packages) in `~/linux` , the top-level directory relative to the kernel version's directory on the filesystem. Install them via:

    sudo dpkg -i ~/linux/*.deb

For firmware, run this to build and install the latest blobs from upstream:

    cd ~/linux
    git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/
    cd linux-firmware
    sudo make install -j$(nproc)

Trigger the rebuild of all DKMS modules by running:

    sudo su
    
    ls /usr/src/linux-headers-* -d | sed -e 's/.*linux-headers-//' | \
    
    sort -V | tac | sudo xargs -n1 /usr/lib/dkms/dkms_autoinstaller start

When done, you can reboot and test your new kernel:

    sudo systemctl reboot

Fin.




