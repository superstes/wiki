*************
Extend Drives
*************

.. warning::

  If you don't know what you are doing => you should not make changes like these on an important system!

  You might **BREAK YOU SYSTEM**!!

  Try it on a useless VM and play around with it or leave it to the pros.

Linux
#####

These examples will go into how to extend volumes and/or drives on a linux VM.

LVM
***

Details
=======

We need to go through these steps:

#. Extend the base drive or partition

  #. Drive

  #. Partition

    .. code-block:: bash

        # start fdisk targeting the modified disk
        fdisk /dev/sdX
        # list partition layout
        p
        # delete partition
        d
        => number of partition to increase
        # re-create the partition, but bigger
        n
        => enter or choose custom parition number
        => enter
        => enter
        => remove the Signature?
        Y
        # modify parition type
        t
        30  # for LVM disk
        # verify
        p
        # save and write
        w

2. Resize the LVM physical device

    .. code-block:: bash

       pvresize /dev/sdX

#. Extend the LVM volume group

    .. code-block:: bash

       vgextend vg0 /dev/sdX

#. Extend the LVM logical volume

    .. code-block:: bash

        lvextend /dev/vg0/lv1 -L 20GB

#. Update the parition size

    .. code-block:: bash

       resize2fs /dev/mapper/vg0-lv1

Script
======

.. code-block:: bash

    #!/bin/bash
    PD='sda'
    LVM_PV="${PD}2"
    LVM_VG='vg0'
    LVM_LV='lv1'
    EXT='20GB'

    fdisk "/dev/${PD}"
    pvresize "/dev/${LVM_PV}"
    vgextend "${LVM_VG}" "/dev/${LVM_PV}"
    lvextend "/dev/${LVM_VG}/${LVM_LV}" -L "${EXT}"
    resize2fs "/dev/mapper/${LVM_VG}-${LVM_LV}"
