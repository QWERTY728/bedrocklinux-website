Title: Bedrock Linux 0.7 Poki Known Issues
Nav: poki.nav

Bedrock Linux 0.7 Poki Known Issues
===================================

Issues listed here are supplemental to [issues listed in the compatibility section](compatibility-and-workarounds.html).

## {id="lvm"} lvm mount failures

Some users have reported issues with lvm partitions, most notably `/home`, not mounting.

Typical init systems do not mount `/etc/fstab` values corresponding to ~{global~} directories such as `/home`, and thus Bedrock is required to do so itself.  However, Bedrock does not currently know how to populate `/dev/mapper` files required for lvm.

A future Bedrock update may embed `dmsetup` into Bedrock to allow it to populate `/dev/mapper` and mount ~{global~} lvm partitions such as `/home`.

## {id="x11-repeated"} /bedrock/cross/bin/X11/

The `/bedrock/cross/bin/X11` directory recursively contains many `X11` directories.

This is because many distros contain a symlink at `/usr/bin/X11` which points to `.` which `/bedrock/cross/bin` tries to expand.

A possible fix for this would be for `cross-bin` to ignore directories.

## {id="fuse-sigterm"} etcfs and crossfs sigterm handling

Bedrock Linux has two FUSE filesystems, etcfs and crossfs.  Ideally, both should unmount themselves on SIGTERM, such as when the system is shutting down.  Currently they do not do so.  This may cause problems with sysv inits which have shutdown scripts within `/etc`.

## {id="unmount-warnings"} Unmount warnings on shutdown

Bedrock uses a Linux kernel feature which propagates some mount and unmount operations.  When an init system performs an umount operation on shutdown, this may actually unmount multiple mount points.  Some init systems are confused by this, as it is not a commonly used feature, and print warnings about being unable to unmount directories because they already unmounted them.  These warnings are harmless aesthetic issues.  Note that this does not mean *all* warnings about mount difficulties on shutdown are harmless and only refers to a specific subset.

## {id="fsck-root"} Root filesystem fsck may be skipped

Due to a quirk in how Bedrock works, init systems may not mount the root directory read-only in preparation for providing it to `fsck`.  Bedrock attempts to disable this by changing the corresponding field in `/etc/fstab`.

Some initrds will `fsck` the root filesystem, but not all.  Bedrock should offer the option of calling `fsck` on the root filesystem itself.  However, it currently does not.

## {id="localegen-single"} Bedrock localegen only understands single value

Bedrock has a `localegen` field in `bedrock.conf` which, if populated, will be used to configure the locale of fetched strata.  However, it only understands a single value.  This should be expanded to support multiple localegen values.

Moreover, research should be performed into the viability of making `/etc/locale.gen` global.  This would remove the need for Bedrock to understand this field.
