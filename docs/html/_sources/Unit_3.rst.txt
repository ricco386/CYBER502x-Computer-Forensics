.. _unit3:

Unit 3 - Unix/Linux File System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _unit3_booting_process:

Booting Process
---------------

.. _mbr:

The BIOS and MBR partitioning scheme was developed in the early 1980's by IBM Personal Computers. The basic
input/output system short for BIOS starts power on self-test. Searches, loads, and executes the boot loader following
the BIOS boot sequence.

Commonly, BIOS loads and executes the master boot record, boot loader. MBR is always located in sector zero of
the drive and it contains the initial boot code and the disk partition information.

MBR is always 512 bytes and it is created when the disk is partitioned. The MBR contains 446 bytes of executable code
called master boot code. Two bytes of the MBR signature and the 64-bytes partition table information.

The MBR signature is always set to the hex 55AA, which marks the end of MBR. If you add those bytes together,
446 plus 64 plus 2, you will end up get 512 bytes in total.

The disk signature, a unique number at offset 01B8 identifies the disk to the operating system.

The partition table contains four primary partition entries. Each partition entry uses 16 bytes to specify the starting
and ending position in cylinder-head-sector for each partition as well as the active flag indicating whether
the partition is active for booting or not.

Once it has identified the active partition, MBR will load a copy of the boot sector from the active partition into
memory and transfer control to the executable code in the boot sector of the active partition.

.. _efi:

In the 1990's, Intel introduced EFI boot framework. EFI stands for extensible firmware interface to especially support
larger disk spaces. EFI uses globally unique identification partition known as GPT and jumps to EFI system partition
to begin the initial boot strapping.

While MBR can only have four primary partitions, GPT can support up to 128 primary partitions on a GPT disk.

.. _unit3_linux_file_systems:

Linux/Unix File Systems
-----------------------

When a partition has a file system installed, the partition has an organized structure for files. This is basically
what the file system is for.

.. _ext_filesystem:
.. _superblock:

Linux EXT file systems are derived from the UNIX file system. When an EXT file system is installed, the partition
contains an optional boot block and superblock. The superblock defines the data structures and the boundaries of this
file system.

The superblock defines the data structures and the boundaries of this file system. The information in superblock
includes block size, in Linux UNIX file systems, a block is the smallest unit to store information. Like the block size,
the number of blocks in a group is fixed when the file system is created.

It also includes block and INODE bitmap. The bitmap contains zero and one. One means the block or INODE is in use.
The number of free blocks and the number of free INODE in the file system is also recorded in superblock.

.. note::

    If the block size is 1024 bytes, a file with 8 bytes will have to use one block.

.. _inode:

The first INODE in EXT2, 3, 4 file system is the INODE for the system root directory. It is clear that data is saved
in blocks, but what is really INODE for? INODE contains metadata information about a file.

In Linux UNIX systems, each file has a correspondent INODE. However, multiple files may share a same INODE. **The number
of files pointed to the same INODE is called a link count.** The metadata stored in INODE includes file type, owner,
file permissions, file's last modification, access, and INODE changing times, link count, and the file's data block
addresses that store the actual file contents.

The content of a directory is a list of directories representing files and subdirectories that reside in this directory.
Each entry contains a file directory name and its correspondent item number. Therefore, the mapping information between
INODE number and the file name is stored in a directory.

Directory example:


+--------------------------+--------------+-----------+
| Byte offset in directory | Inode number | File name |
+--------------------------+--------------+-----------+
| 0                        | 70           | .         |
+--------------------------+--------------+-----------+
| 16                       | 35           | .\.       |
+--------------------------+--------------+-----------+
| 32                       | 123          | file1     |
+--------------------------+--------------+-----------+
| 48                       | 345          | file2     |
+--------------------------+--------------+-----------+
| 64                       | 90           | file3     |
+--------------------------+--------------+-----------+

Walkthrought to locate content of the file `/etc/myconfig`

The file names start with slash. From the superblock, we have the information of the first INODE number correspondent
to slash. Let's assume it is INODE zero.

.. sourcecode::

     -----------------
     | inode 0 ... / |
     |               |
     |               |
     |               |
     -----------------

We read the information stored in INODE zero, since it is a directory, you will find out the INODE type is directory
and you will get data block addresses for slash for INODE zero's content.

Go to slash data block addresses. It contains the list of files and subdirs contained in slash. Subdir etc along with
its correspondent INODE number should be in this list. If not, it will display an error message of cannot locate
`/etc/myconfig`.

.. sourcecode::

     -----------------         -----------------------
     | inode 0 ... / |--- / ---| inode 12 ... foo    |
     |               |         | inode 13 ... etc    |
     |               |         | inode 14 ... db.csv |
     |               |         | ...                 |
     -----------------         -----------------------

Since `/etc` is a directory, you will find out the INODE type is directory and then you will get the data block
addresses for `/etc`.

Go to the data block addresses of `/etc`. Because it is a directory, it contains a list of files and subdirectories
that reside in `/etc`. File name `myconfig` along with its correspondent INODE number should be in this list. If not,
again, error message will be displayed.

.. sourcecode::

     -----------------         -----------------------           --------------------------
     | inode 0 ... / |--- / ---| inode 12 ... foo    |           | inode 20 ... misc      |
     |               |         | inode 13 ... etc    |--- etc ---| inode 21 ... stuff.txt |
     |               |         | inode 14 ... db.csv |           | inode 22 ... myconfig  |
     |               |         | ...                 |           | ...                    |
     -----------------         -----------------------           --------------------------

Knowing the INODE number of `/etc/myconfig`, we will read this INODE's content. Since `/etc/myconfig` is a regular file,
you will find out the INODE type is file. And you will get the data block addresses for this file. Go to the data
block addresses of `/etc/myconfig`. Because it is a regular file, we will have the file content in the data block.

.. sourcecode::

     -----------------------           --------------------------                ----------------
     | inode 12 ... foo    |           | inode 20 ... misc      |                | file block 1 |
     | inode 13 ... etc    |--- etc ---| inode 21 ... stuff.txt |                | file block 2 |
     | inode 14 ... db.csv |           | inode 22 ... myconfig  |--- myconfig ---| file block 3 |
     | ...                 |           | ...                    |                | ...          |
     -----------------------           --------------------------                ----------------

.. note::

    Recovering a deleted EXT3 file is difficult, due to the way EXT file system locates file content based on a file name.

Can we retrieve the deleted file even though the OS will not help us?

The answer varies depending on file systems implementations. For example, Microsoft's FAT file system will mark the file
as deleted by renaming the file name. The Berkeley Fast File System usually breaks all the connections between
directory entry and the file data blocks.

**When we create a new file on EXT file system, each file has its content stored in blocks, its metadata stored in
INODE, and its file name and INODE mapping stored in its parent directory.** When a new file (not a hotlink) is created,
a free INODE is chosen from the INODE bitmap. The superblock free INODE values are decremented by one. An entry is
added in the parent directory. Free data blocks are chosen from the block bitmap to store the file content. Finally,
the INODE content is viewed.

**When a file is deleted in EXT2 file system, the data blocks in the block bitmap are marked as free. The INODE in
INODE bitmap is marked as free. The director entry is marked as unused.** The connections among directory entry, INODE,
and the file data blocks will still be there until they are overwritten.

Therefore, if the directory entry information is still available, you still can find the mapping between the file name
and its INODE. If the INODE has not been overwritten yet, you can even find the file's metadata, including permissions
and data blocks. Therefore, even though the file has been deleted, it is still possible to recover the file content
if the data blocks are not overwritten.

For EXT3 and EXT4 file systems, the file size in INODE is set to zero. More destructively, the data block information
in the INODE is cleared upon file deletion, which means we are not able to link the INODE to the file content even if
the content is still intact. **Data recoveries from EXT3 and 4 file systems are harder, but may still be possible.**

.. note::

    Deleted data still exists on disk and are recoverable until space is overwritten. In addition, larger disks are less
    likely to overwrite formerly used space unless the disk blocks are wiped after file is deleted.

.. _srm:
.. _shred:

Both Linux,UNIX commands `srm <https://en.wikipedia.org/wiki/Srm_(Unix)>`_ or
`shred <https://en.wikipedia.org/wiki/Shred_(Unix)>`_ will intentionally destroy file content.

.. _sleuthkit:
.. _autopsy:

Sleuthkit and Autopsy
---------------------

Sleuth Kit supports almost all file systems. Autopsy is the GUI front end for Sleuth Kit. Sleuth Kit view the
file system in five layers and it contains tools for each layer except for the physical layer.

The physical layer
==================

It uses magnetic hard disks and then solid state drives to physically store data.

Magnetic hard disks operate by creating or detecting magnetic fields in fixed regions or blocks of a magnetic surface.
When the disk is asked to write to a particularly block, it spins and moves its head to a certain location and
magnetize the magnetic surface of the disk in that region. If there is already old discarded data in that location,
the data is automatically converted or overwritten to the new values.

Solid state drives, on the other hand, have no moving mechanical components. Data is stored in fixed arrangements of
electronic transistors. A couple of SSD properties will affect forensic analysis.

Data rewrite requires blocks to be erased electronically before they can be used again. The write over old data
property of magnetic disks does not work in SDD. To increase performance for writing, a technology called garbage
collection was built into SSD computers to help automatically reset the used data blocks back to free space. The
garbage collection process will erase old data, so recovering deleted data will be affected to some degree in SDD.

The data layer
==============

It uses blocks or clusters to store data. Cluster is a Windows term for block.

The metadata layer
==================

It contains a file's metadata information. For example, INODE information.

The file system layer
=====================

It describes the file system structure details such as information in super block.

The file name layer
===================

It associates a file name to its metadata structure.

We can break Sleuth Kit into four sets of tools:

 * File system layer tools start with FS.
 * File name layer tools start with F.
 * Metadata layer tools start with I.
 * Data layer tools start with BLK, which is block.

**TODO: Video commands**