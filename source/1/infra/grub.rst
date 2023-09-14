.. _infra_grub:

.. include:: ../../_inc/head.rst

.. warning::

    It has been some time since I've set-up a system this way. (*as of writing this*)

    The information be missing some details.

****
GRUB
****

Redundant boot disks
####################

EFI boot does not yet support software-raid (MD) => see: `Debian documentation - UEFI Grub <https://wiki.debian.org/UEFI#RAID_for_the_EFI_System_Partition>`_

After installing the system with a single boot-partition, we will have to reinstall it on the second one!

Disk
****

You will have to use a second disk to get redundancy.

Create a boot partition at its beginning and mark it as bootable! 512MB should be enough.

Reinstall
*********

See also `Debian documentation - Grub reinstall <https://wiki.debian.org/GrubEFIReinstall>`_

First we will have to boot from a live-system! Example `Debian Live-system <https://www.debian.org/CD/live/>`_

After that we can install grub on the second disk:

.. code-block:: bash

    # sda is the new disk
    mount /dev/sda3 /mnt
    mount /dev/sda2 /mnt/boot
    mount /dev/sda1 /mnt/boot/efi
    mount --rbind /dev  /mnt/dev
    mount --rbind /proc /mnt/proc
    mount --rbind /sys  /mnt/sys
    chroot /mnt
    grub-install /dev/sda --efi-directory=/boot/efi --target=x86_64-efi

Make sure to enable both of the disks in the UEFI/BIOS boot sequence.

Sync
****

After installing redundant boot-partitions - we still have a problem when doing a system update.

It will only update the kernel version on the currently active partition!

To fix this we can:

* Mount both boot partitions in the host system

  .. code-block:: bash

      # find disk ID's
      ls -l /dev/disk/by-id/

      cat /etc/fstab
      > ...
      > /dev/disk/by-id/wwn-0x5001b444a60ff504-part1                       /boot/efi vfat defaults,noatime 0 2
      > /dev/disk/by-id/wwn-0x5001b444a60ff504-part2                       /boot     ext4 defaults 0 1
      > /dev/disk/by-id/ata-SanDisk_X400_M.2_2280_256GB_170839425792-part1 /boot/efi vfat defaults,noatime 0 2
      > /dev/disk/by-id/ata-SanDisk_X400_M.2_2280_256GB_170839425792-part2 /boot     ext4 defaults 0 1

* Add a sync job:

  .. code-block:: bash

      # todo: add script
