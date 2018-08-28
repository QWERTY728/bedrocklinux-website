Title: Bedrock Linux 1.0beta2 Nyla Stratum Setup Instructions
Nav: nyla.nav

Bedrock Linux 1.0beta2 Nyla Stratum Setup Instructions
======================================================

These are instructions for installing other Linux distributions as ~{strata~}
within Bedrock Linux 1.0beta2 Nyla.

See the [tips, tricks and troubleshooting](troubleshooting.html) page after
installing each of these for other advice about using the specific distribution
as a ~{stratum~}.

- [Any Linux Distribution](#any)
- [Debian-based Linux distributions](#debian-based)
- [Arch Linux](#arch)
- [Yum-based distros (Fedora, Centos, Scientific Linux, OpenSUSE)](#yum-based)
- [Gentoo Linux](#gentoo)
- [Void Linux](#void)
- [CRUX](#crux)
- [Alpine Linux](#alpine)

## {id="any"} Any Linux Distribution

If there are no instructions below specific to a Linux distribution which you
would like to make into a ~{stratum~} for your Bedrock Linux install, you can
usually fall back to installing the distribution through its normal
installation means. Once it is installed, you may simply copy its root
directory to `/bedrock/strata/~(stratum-name~)`.  When installing the Linux
distribution by its normal means, be very careful when partitioning, and be
careful to avoid having the bootloader take over your system.

For example, if you install Slackware to a USB flash drive, you can mount the
USB flash drive in Bedrock Linux and copy its contents to
`/bedrock/strata/slackware`.

However, this method requires rebooting as well as raises the possibility of
unintentionally wiping something important when partitioning or forcing you to
reinstall your bootloader, and thus the distro-specific instructions described
below may be preferable if available.

You may also be able to install a ~{stratum~} distribution in a virtual machine
which you can mount and copy the files out, or you can use a scripts or tools
used to build containers such as LXC.

## {id="debian-based"} Debian-based Linux distributions

The essentials of Debian-based Linux distributions can be installed through a
program called `debootstrap.`  Debootstrap is a shell script which can be
easily installed into most Debian-based Linux distributions, and is often
available in the repositories of non-Debian-based Linux distributions, such as
Fedora. While it is possible to install `debootstrap` (by first installing dpkg
and pkgdetails) into just about any other Linux distribution as well, it is not
covered here. Busybox's dpkg does not seem sufficient for `debootstrap`.

Boot into a Linux distribution which can run `debootstrap`, or use an existing
~{stratum~} which can use `debootstrap` in Bedrock Linux if available.
LiveCD/LiveUSBs such as Knoppix or an Ubuntu installer should work.

Ensure the pre-requisites for `debootstrap` are available. This can be done by
installing `debootstrap` through the distribution's package manager (which should
bring in its dependencies) if available. Next, download the .deb file for the
`debootstrap` specific to ~{stratum~} Linux distribution release you would like, or a
newer `debootstrap` .deb from the same distribution. For example, for Debian
Jessie, grab the file made available from
[here](http://packages.debian.org/jessie/debootstrap). If you are attempting
to use `debootstrap` from a non-debian-based Linux distribution, convert the .deb
file to the native package format with something such as the `alien` package.

Install the package. If on a debian-based system (as root):

	{class="rcmd"} dpkg -i debootstrap_~(VERSION~).deb

Make a directory in which to put the target ~{stratum~} Linux distribution.  If you
are doing this from something other than Bedrock, such as a LiveUSB/LiveCD, be
sure to mount the appropriate partition which you would like to contain your
~{stratum~} and create the directory in there.

	{class="rcmd"} mkdir /bedrock/strata/~(stratum-name~)

Use debootstrap to download and set up the target ~{stratum~} Linux distribution.

	{class="rcmd"} debootstrap --arch ~(architecture~) ~(release~) ~(path~) ~(repository~)

For example, to install the (64-bit) x86\_64 Debian jessie to
`/bedrock/strata/jessie` using `http://ftp.us.debian.org/debian`:

- {class="rcmd"}
- debootstrap --arch amd64 jessie /bedrock/strata/jessie http://ftp.us.debian.org/debian

It may take a bit to download and unpackage the various components.

Check to see if it created a non-blank `/var/lib/dpkg/statoverride` file, and
if it did, delete the content (ie, leave a blank file in its place).  See [this
troubleshooting item](troubleshooting.html#statoverride).

- {class="rcmd"}
- echo -n "" > /bedrock/strata/~(stratum~)/var/lib/dpkg/statoverride

Don't forget to edit `/etc/apt/sources.list` and other ~{stratum~}-specific settings.

Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(jessie~)]
    framework = default
    init = /lib/systemd/systemd

## {id="arch"} Arch Linux

There are three strategies to acquiring an Arch Linux ~{stratum~} at this point
in time.  Follow any of the methods below to acquire the files for Arch Linux,
placing them into `/bedrock/strata/~(arch~)`.  Once you have done so, you may
still have to set up `pacman` - continue reading below.

- The archlinux-bootstrap tarballs.  Go to your favorite [Arch Linux
  mirror](https://www.archlinux.org/mirrors/status/) and look in the
  `iso/latest/` directory.  You should see two tarballs - one for i686 and the
  other for x86\_64.  Download and untar the one you want.  It will give you a
  "root.~(arch~)" directory - more/rename this to the name/location of the
  ~{stratum~} you want.
- The archbootstrap script, available [here](https://github.com/tokland/arch-bootstrap).
- The
  [arch-install-scripts](https://www.archlinux.org/packages/?name=arch-install-scripts)
  package contains a `pacstrap` script which can be used to bootstrap a Arch
  Linux system.  This is useful if you already have an Arch Linux system on
  hand to bootstrap another one.  Once you have `pacstrap`, you can install the
  arch ~{stratum~} with `{class="rcmd"} pacstrap -d /bedrock/strata/~(arch~) base
  base-devel`

Once you have the files, you may still have to setup `pacman`'s keys.  Chroot
into the ~{stratum~}:

- {class="rcmd"}
- export STRATUM=~(arch~)
- cp /etc/resolv.conf /bedrock/strata/$STRATUM/etc
- mount -t proc proc /bedrock/strata/$STRATUM/proc
- mount -t sysfs sysfs /bedrock/strata/$STRATUM/sys
- mount --bind /dev /bedrock/strata/$STRATUM/dev
- mount --bind /dev/pts /bedrock/strata/$STRATUM/dev/pts
- mount --bind /run /bedrock/strata/$STRATUM/run
- chroot /bedrock/strata/$STRATUM /bin/sh

Run the following commands to setup `pacman`.  It may speed things up to use your
mouse and keyboard to help generate entropy.

- {class="rcmd"}
- pacman-key --init
- pacman-key --populate archlinux

When you have finished, run the following to clean up:

- {class="rcmd"}
- exit   #(to leave the chroot)
- umount /bedrock/strata/$STRATUM/proc
- umount /bedrock/strata/$STRATUM/sys
- umount /bedrock/strata/$STRATUM/dev/pts
- umount /bedrock/strata/$STRATUM/dev
- umount /bedrock/strata/$STRATUM/run

Edit the following two files to configure `pacman` to your liking:

- /bedrock/strata/~(arch~)/etc/pacman.d/mirrorlist
- /bedrock/strata/~(arch~)/etc/pacman.conf

Remove/comment out "CheckSpace" from
`/bedrock/strata/~(arch~)/etc/pacman.conf`.

Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(arch~)]
    framework = default
    init = /lib/systemd/systemd

## {id="yum-based"} Yum-based distros (Fedora, Centos, Scientific Linux, OpenSUSE)

There is a utility called `rinse` which can be utilized to acquire various
`yum`-based distros.  Acquire `rinse`, either [from its
website](http://collab-maint.alioth.debian.org/rinse/) (then manually
acquiring its dependencies) or from a ~{stratum~}'s repository (e.g. Debian has
it, Arch has it in AUR, etc).  Then use it to acquire the desired files.

- {class="rcmd"}
- rinse --list-distributions # find the desired release name
- rinse --arch amd64 --distribution ~(centos-7~) --directory /bedrock/strata/~(centos7~)


Then create an entry in `/bedrock/etc/strata.conf` file as explained in [the
configuration page](configure.html), such as:

    [~(centos7~)]
    framework = default
    init = /lib/systemd/systemd

## {id="gentoo"} Gentoo Linux

Gentoo Linux provides a tarball of the userland, which makes installing it as a
Bedrock Linux ~{stratum~} fairly simple. Note that this is a quick overview of
the steps required in getting Gentoo working as a Bedrock Linux ~{stratum~}.
For more information on configuring and using Gentoo, consult the [Gentoo
Handbook](http://www.gentoo.org/doc/en/handbook/).

To download the tarball, navigate to the
[Gentoo mirrorlist](http://www.gentoo.org/main/en/mirrors2.xml) and choose
the mirror that is closest to you. Once you've followed the link to the mirror,
navigate to `releases/amd64/autobuilds/current-stage3-~(arch~)` for 64-bit, or 
`releases/x86/autobuilds/current-stage3-~(arch~)` for 32-bit, and download the
appropriate stage3 tarball to the directory that Gentoo is being installed into.

Unpack the tarball.

- {class="cmd"}
- tar -xvjpf stage3-\*.tar.bz2 # Note the "-p" option

The next step is to configure `~(/bedrock/strata/gentoo~)/etc/portage/make.conf` file
so that you can compile the appropriate utilities using portage. For information
on how to optimize portage for comiplation on your machine, consult Gentoo's
[Compilation Optimization Guide](http://www.gentoo.org/doc/en/gcc-optimization.xml).

After configuring your compilation optimization variables, it is time to set up
the system so that you can chroot into it to finish the installation process.

- {class="rcmd"}
- export STRATUM=~(gentoo~)
- cp /etc/resolv.conf /bedrock/strata/$STRATUM/etc
- mount -t proc proc /bedrock/strata/$STRATUM/proc
- mount -t sysfs sysfs /bedrock/strata/$STRATUM/sys
- mount --bind /dev /bedrock/strata/$STRATUM/dev
- mount --bind /dev/pts /bedrock/strata/$STRATUM/dev/pts
- mount --bind /run /bedrock/strata/$STRATUM/run
- chroot /bedrock/strata/$STRATUM /bin/sh

You will now install portage while inside the Gentoo chroot.

- {class="rcmd"}
- mkdir /usr/portage # portage will be installed here
- emerge-webrsync # download and install the latest portage snapshot
- emerge --sync # update the portage tree

Now, before installing anything with Gentoo, it is recommended that you choose
a system profile. This will set up default values for your `USE` variable, among
other things. You can view the available profiles with

- {class="rcmd"}
- eselect profile list

and set it by selecting the number associated with the desired configuration

- {class="rcmd"}
- eselect profile set ~(PROFILE~)

Finally, you may configure your `USE` flags in `/etc/portage/make.conf`. `USE`
flags are one of the most powerful features in Gentoo. They are keywords that
allow you to tell portage what dependencies and you would like to allow or
block from your system. For information on how to use `USE` flags, consult the
[USE flags](http://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=2&chap=2)
section of the Gentoo Handbook.

It is recommended that you update your system to be compatible with your newly
configured `USE` flags. However before recompiling your system, you may want to
emerge `gentoolkit`, which provides the `revdep-rebuild` utility. This will
allow you to rebulid the applications that were dynamically linked to the
now-removed software but don't require them anymore.

- {class="rcmd"}
- emerge gentoolkit
- emerge --update --deep --with-bdeps=y --newuse world # update entire system
- emerge --depclean # remove orphaned dependencies
- revdep-rebuild # rebuild applications with broken dynamic links

Now that Gentoo is fully set up, exit the chroot and remove the mounts

- {class="rcmd"}
- exit   #(to leave the chroot)
- umount /bedrock/strata/$STRATUM/proc
- umount /bedrock/strata/$STRATUM/sys
- umount /bedrock/strata/$STRATUM/dev/pts
- umount /bedrock/strata/$STRATUM/dev
- umount /bedrock/strata/$STRATUM/run


Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(gentoo~)]
    framework = default
    init = /sbin/init

## {id="void"} Void Linux

Void Linux provides a static version of its package manager, xbps, which can be
used to bootstrap a ~{stratum~}.

First, download a static build of xbps for your CPU architecture.  Links to xbps-static for various architectures are listed [here](http://www.voidlinux.eu/usage/xbps/#download-static-binaries).

Then, extract it into its own directory

- {class="cmd"}
- mkdir xbps-static
- cd xbps-static
- tar xvf ~(/path/to/xbps-static-tarball~)

Next, as root, tell it to acquire the files:

- {class="rcmd"}
- usr/bin/xbps-install -S -R http://repo.voidlinux.eu/current -r /bedrock/strata/~(void~)/ base-system

Clean up:

- {class="rcmd"}
- cd ..
- rm -rf ./xbps-static

Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(void~)]
    framework = default
    init = /sbin/init

## {id="crux"} CRUX

CRUX's installation ISO includes a static build of the CRUX `pkgadd` utility.
This can be used to install the CRUX system from packages also included on the
ISO.

First, download the installation ISO [from one of the mirrors](https://crux.nu/Main/Download).

Next, as root, mount it so that the files are accessible:

- {class="rcmd"}
- mkdir -p ~(/mnt/crux-mount~)
- mount -o loop,ro crux-~(VERSION~).iso ~(/mnt/crux-mount~)

Then extract the package manager:

- {class="cmd"}
- tar xvf ~(/mnt/crux-mount~)/crux/core/pkgutils\*.tar\* usr/bin/pkgadd -O > ~(./pkgadd~)
- chmod a+rx ~(./pkgadd~)

Make some required files and directories in preparation for pkgadd.

- {class="rcmd"}
- mkdir -p /bedrock/strata/~(crux~)/var/lib/pkg
- touch /bedrock/strata/~(crux~)/var/lib/pkg/db

Install the packages:

- {class="rcmd"}
- for pkg in ~(/mnt/crux-mount~)/crux/core/\*; do echo "installing $pkg" && ~(./pkgadd~) -r /bedrock/strata/~(crux~)/ $pkg; done

Clean up:

- {class="rcmd"}
- umount ~(/mnt/crux-mount~)
- rmdir ~(/mnt/crux-mount~)
- rm ~(./pkgadd~)

Configure CRUX's `/etc/rc.conf` at `/bedrock/strata/~(crux~)/etc/rc.conf`.  Be
sure to set the `TIMEZONE` to `/bedrock/etc/localtime` and `HOSTNAME` to the
desired hostname.

Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(crux~)]
    framework = default
    init = /sbin/init

## {id="alpine"} Alpine Linux

Alpine Linux provides a static version of its package manager, apk, which can
be used to bootstrap a ~{stratum~}.

First, acquire a static copy of the apk from one of the
[mirrors](http://nl.alpinelinux.org/alpine/MIRRORS.txt).  The package will be
available at
~(mirror-url~)/latest-stable/main/~(arch~)/apk-tools-static-~(version~).apk

Extract the package manager:

- {class="cmd"}
- tar xvf apk-tools-static-~(version~).apk sbin/apk.static -O > ~(./apk~)
- chmod a+rx ~(./apk~)

Next, as root, tell it to acquire the files:

- {class="rcmd"}
- ~(./apk~) -X ~(mirror-url~)/latest-stable/main -U --allow-untrusted --root /bedrock/strata/~(alpine~)/ --initdb add alpine-base

Configure apk:

- {class="rcmd"}
- echo "~(mirror-url~)/latest-stable/main" > /bedrock/strata/~(alpine~)/etc/apk/repositories

Clean up:

- {class="rcmd"}
- rm ~(./apk~)

Enable desired openrc services, such as hostname:

- {class="rcmd"}
- chroot /bedrock/strata/~(alpine~) rc-update add hostname default

Finally, create an entry in `/bedrock/etc/strata.conf` file as explained
in [the configuration page](configure.html), such as:

    [~(alpine~)]
    framework = default
    init = /sbin/init
