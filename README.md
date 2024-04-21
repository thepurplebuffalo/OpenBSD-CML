# OpenBSD-CML
Instructions for building an OpenBSD node in Cisco Modeling Lab

## Why OpenBSD in CML?

OpenBSD is standards-compliant, has good documentation, and is secure.

OpenBSD can act as a router, a routed layer 3 firewall, a NAT boundary, a
bridging Layer-3 firewall, an end host, a webserver, a load balancer, and many
other roles not mentioned here.

# Create a qcow2 OpenBSD VM

As resources in CML are generally scarce I suggest starting with a fairly
minimal configuration for a VM:

* RAM: 512MB
* Disk: 8GB
* CPU: Single
* Network: single interface with Internet access

Although I used VMWare to build this image it should be equally difficult or
easier to use other platforms (VirtualBox, QEMU, KVM, etc).

I used the amd64
[install75.iso](https://cdn.openbsd.org/pub/OpenBSD/7.5/amd64/install75.iso)
image to perform a base install.  I did not install games nor any of the x11
packages (just to save space).

I used ```cisco``` for the root password.  I disabled root SSH logins (habit).

Auto partitioning was sufficient for this.

I created a second user named ```cisco``` with the same password.

## First boot

Once the base system is installed reboot and log in as root.

Right away you should check for updates:

    syspatch
    pkg_add -u

### Install additional packages

These packages have been extremely useful through the years.

    pkg_add mtr-- wget gnuwatch nmap wireguard-tools

### Add username and password hints

If some other admin tries to use your newly minted node it would be nice if
they can easily see what username and password to use.  This will appear on
the serial console.

Update the ```/etc/gettytab``` file.  Replace the existing default section with
this:

    default:\
            :np:im=\r\n%s/%m (%h) (%t)\r\n\
    ************************\r\n\
    * Console login        *\r\n\
    * username = root      *\r\n\
    * password = cisco     *\r\n\
    *                      *\r\n\
    * General login (SSH)  *\r\n\
    * username = cisco     *\r\n\
    * password = cisco     *\r\n\
    ************************\r\n\
    \r\n:sp#1200:

### Enable IP forwarding

Enable IP forwarding.  By default this is turned off.  This isn't strictly
required for an end host nor webserver; but it's a step easily missed.

    echo "net.inet.ip.forwarding=1" > /etc/sysctl.conf

### Enable logins on serial console

Enable serial console to provide a login prompt
Edit ```/etc/ttys``` so that ```tty00``` reads as:

    tty00   "/usr/libexec/getty std.9600"   vt220   on  secure

NB: you can output to a serial console without being able to login on it.  This
step enables that login.

## Snapshot

Now is a good time to create a snapshot of your VM. This point in the install
it's still fairly easy to install patches and additional software.

## Finalize the image

This is the last step before exporting your image to a qcow2 file.  These
commands will enable output to the serial console needed for CML and cleanup
the original network interface.  It's hard to undo these (but not impossible).

    rm /etc/hostname.em0
    echo "set tty com0" > /etc/boot.conf
    shutdown -p now

Now you can export your disk image for use in CML.

## (Optional) Convert VMWare to qcow2

This step is not required if you already have a qcow2 image.  VMWare provides
a series of VMDK files which need to be converted.

Copy all of the VMDK files from your image to someplace where you can run the
```qemu-img``` tool (available in Linux).

    qemu-img convert -f vmdk -O qcow2 <base filename.vmdk> <target qcow2 filename>

If you receive an error you may be specifying the wrong base VMDK file.  Look
for the smallest VMDK file and use it.

# CML Node Definition



# CML Image Definition


