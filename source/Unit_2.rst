.. _unit2:

Unit 2 - Linux/Unix Acquisition 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _unit2_acquisition_preparation_and_system_information_acquisition:

Acquisition Preparation and System Information Acquisition
----------------------------------------------------------

Linux/Unix forensics follows the general forensic process of **collecting, preserving, analyzing, and reporting evidence**.

1. First is a set of forensic tools, including a bootable live CD.
   
   The reason is simple: You should never trust any tools from the suspect machine since it has potentially compromised.
   When booting using a forensic bootable CD in forensic mode, the internal suspect hard drive is not touched.

2. Second is a powerful machine, sometimes we call it forensic machine or evidence server.

   This system has forensic tools installed and its drives are wiped clean for storing acquired evidence.


.. _unit2_volatile_data_acquisition:

Volatile data acquisition
-------------------------

Volatile evidence is evidence that can easily be changed. Every memory dump will be different since memory is constantly changing. That's why we should first acquire the data that is most volatile.

A possible order of collection from most volatile to least volatile until non-volatile is memory; swap space or page file; network status; connections; processes running; open files; drives, and removable media.

Since you are directly collecting volatile evidence from a suspect machine, you are inevitably changing data. To ensure a minimal impact to the original data, you should always try to use small footprint tools and 
you should always document everything you have done.

.. _unit2_collect_information_from_live_system:

Collect information from a live system
======================================

On potentially compromised system we cannot trust anything, any tool from this machine. Use trusted tools from somewhere else, eg. a mounted USB. 

.. note::

    When we run any commands the output has to be saved to outside from potentially compromised machine, eg. a mounted USB.

On the beging of collecting evidence we store date, how long is the machine booted, logged in users and some basic information about the machine

.. sourcecode::

    $ date
    St jan 29 23:07:00 CET 2020
    $ uptime
     23:07:27 up 44 min,  2 users,  load average: 0,00, 0,02, 0,30
    $ uname -a
    Linux l430 5.3.16-300.fc31.x86_64 #1 SMP Fri Dec 13 17:59:04 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
    $ w
     18:06:00 up 21 days, 23:14,  1 user,  load average: 1,84, 1,27, 1,11
    USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
    ricco    tty2      08jan20 21days  0.05s  0.05s /usr/libexec/gnome-session-binary


.. _network_interface_promisc_mode:

Check the network interfaces information, and find out if any is switched to **PROMISC mode**. In promisc mode and it is listening for all the traffic passing through,
even the :ref:`packets <cyber501x:unit4_packets_routing>` is not destinate to this machine it will also collect. A compromised machine, might have promisc mode, so it will try to listening for everything it's passed into this network.

.. sourcecode::

    $ ifconfig

.. _command_netstat:

List network connections and the :ref:`cyber501x:unit5_ports` that are open:

.. sourcecode::

    $ netstat -tulpen

.. _command_ps:

List processes running on the system:

.. sourcecode::

    $ ps -eaf

.. _command_lsof:

List of open files (with multiple variants):

.. sourcecode::

    $ lsof -i 4  # list of open files all files opened with network connection 
    $ lsof -p 2918  # list of open files for preocess 2918
    $ lsof +L1 # list of open files with link count less than 1, e.g. deleted files, not visible by ls (deleted), but still in memory

If we want to find UID zero (which is root owned, root owned UID is powerful during the execution time you have the root privilege). And then the permission, the perm, for set UID is 4000 and for set GID is 2000.
And we don't want to see error messages because you will see permission denied and lots of permission denied issues so the output will be cleaner. Investigator will look into this list and try to find any malicious set UID program.

.. sourcecode::

    $ find / -uid 0 -perm 4000 2> /dev/null


.. _unit2_memory_acquisition:

Memory Acquisition
==================

For a host-based memory dump approach, the investigator needs to have physical access to the system. MEMDUMP is a part of the coroner's toolkit, TCT. To overcome memdump limitations, open source tools, Linux Memory Extractor, 
`LiME <https://github.com/504ensicslabs/lime>_`, and Fmem were developed. Both tools will load a kernel module to the system that allows full memory captures.

A commercial tool called F-Response allows examiners to conduct forensic acquisition remotely. F-Response use a pair of dongles, one for the suspect system and another for the forensic system.

We need to have trusted tools available on a suspected machine. Insert mod and then the module name certainly is the `LiME <https://github.com/504ensicslabs/lime>_` module we are interested. 
We want to insert this module into the kernel and the path specifies where this image dump will be reside:

.. sourcecode::

    $ insmod lime-4.2.0-27-generic.ko "path=/mnt/evidence_folder/memory_dump.bin format=padded"

Once the dump is finished, we need to do a cleanup. First remove the lime module from the memory.

.. sourcecode::

    $ lsmod | grep lime
    $ rmmod lime

.. _command_strings:

We'll have `memory_dump.bin` in out safe directory, eg. on a mounted USB. It's a binary file. We want to try a very simple tool called a `strings`, it is able to print out certain lengths of strings, the lengths 
by default is greater not equal to 4 bytes. Lets do a search in the memory dump for the string greater than 8 bytes and starting with the word forensics:

.. sourcecode::

    $ strings -n 8 /mnt/evidence_folder/memory_dump.bin | grep ^forensics


.. _unit2_nonvolatile_data_acquisition:

Nonvolatile data acquisition
----------------------------

.. _unit2_forensic_imaging_of_drives:

Forensic Imaging of Drives
==========================

Be aware that certain types of hard drives and solid state drives may self-destroy any data on the drive when you remove the device or power it off completely. Toshiba is one example.

There are many high speed forensic images in the market. High speed forensic images can copy up to 30 gigabytes per minute and usually have built-in write blocker functionalities. Software-based imaging tools like `FTK Imager` and 
`dd` can also be used to create a bit-stream copy of drives. However, you will need a write blocker to separate the original drives from the imaging software to prevent software from modifying data in original drives. 
Most of these imaging tools will generate the hash value automatically after the imaging is done.

.. _command_dd:

`dd` reads input blocks one at a time from block level device and it puts them into a memory buffer, applies the selected conversions, then outputs from buffer to the desired location, with a default block size of 512 bytes.

To copy our physical device data, we simply use `dd` to move chunks of bits from a source device to a destination device, ignoring the end of file marker. `dd` copies metadata and the data blocks in their entirety. 
`dd` can redirect by pipe to netcat or other applications to send the data to a networked machine.

.. sourcecode::

    $ dd if=/source/file of=/destination/file
    $ dd if=/source/file | nc 192.168.1.2 2222

conv equal to noerror and sync is often used for forensic imaging to skip the unreadable sectors and then continue copying. More specifically, conv equal to noerror will instruct `dd` to pad the bad sectors with zeros
and move on to continue copying the rest of the data.
The sync option instructs `dd` to keep the sectors in the target device aligned with those from the source device. Thus, they data will not be misplaced in the wrong physical location on the destination copy.

.. sourcecode::

    $ dd if=/source/file of=/destination/file conv=noerror,sync

Since `dd` is a simple tool for data duplication, it will not calculate the hash value for the newly generated image. Therefore, after imaging process is complete, it is your responsibility to compute hash values for both 
the original source and `dd` image. Only if the hashes match, your forensic imaging process is done.

If `/source/file` has a bad sector and you use the `conv=noerror` option, you have a complicated and interesting case, as the hash values will be different, due to the padding of zeros.

Besides creating forensic imaging, `dd` has other uses.

You can use `dd` to wiping the drive with all zeros using this command. This process will basically fill your target drive with zeros, overriding any data as it goes. Some sources say that very old hard drives might still contain 
residual data that an electron microscope might pick up after one pass of cleaning. Therefore, the Department of Defense standard for unclassified hard drive dispersion requires three passes over every byte. Some researchers 
even suggest the seven passes of wiping. After wiping, you can reformat drive.

.. sourcecode::

    $ dd if=/dev/zero of=/dev/hdb

examples of using DD for forensic imaging.

In drive to drive imaging, you physically remove the drive from the suspect computer and connect suspected drive to forensics machine with a write blocker. Assume `hdb` is a clean, wiped drive filled with all zero and 
`hdb`'s capacity is larger than slash dev slash `hda`. We can use both `dd` copies below to copy the content.

.. sourcecode::

    $ dd if=/dev/hda of=/dev/hdb

Since the size of `hdb` is larger than `hda`, after `dd` is done, `hdb` will contain the data from the source followed by a bunch of zeros. Therefore, the hash value of `hdb` will not be the same as the hash of `hda`, 
due to those extra zeros.

To obtain the same hash value, you can use `dd` to carve out the number of blocks copied form `hda`, leaving out zeros. Using the second command, since evidence dot `dd` is a file containing exactly the data from `hda`, 
assuming `dd` copied successfully, the hash value of `hda` will be the same as the hash value of the file `evidence.dd`.

.. sourcecode::

    $ dd if=/dev/hda of=/case1/evidence.dd


`dd` has siblings, `sdd`, and `dcfldd`. Both of them improved `dd`'s functionalities by achieving better performance and also providing copy progresses.