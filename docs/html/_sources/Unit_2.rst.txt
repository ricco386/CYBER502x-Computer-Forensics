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
=========================

Volatile evidence is evidence that can easily be changed. Every memory dump will be different since memory is constantly changing. That's why we should first acquire the data that is most volatile.

A possible order of collection from most volatile to least volatile until non-volatile is memory; swap space or page file; network status; connections; processes running; open files; drives, and removable media.

Since you are directly collecting volatile evidence from a suspect machine, you are inevitably changing data. To ensure a minimal impact to the original data, you should always try to use small footprint tools and you should always document everything you have done.

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

List network connections and the :ref:`cyber501x:unit5_ports` that are open:

.. sourcecode::

    $ netstat -tulpen

List processes running on the system:

.. sourcecode::

    $ ps -eaf

List of open files (with multiple variants):

    $ lsof -i 4  # list of open files all files opened with network connection 
    $ lsof -p 2918  # list of open files for preocess 2918
    $ lsof +L1 # list of open files with link count less than 1, e.g. deleted files, not visible by ls (deleted), but still in memory



.. _unit2_memory_acquisition:

Memory Acquisition
------------------

For a host-based memory dump approach, the investigator needs to have physical access to the system. MEMDUMP is a part of the coroner's toolkit, TCT. To overcome memdump limitations, open source tools, Linux Memory Extractor, LiME, and Fmem were developed. Both tools will load a kernel module to the system that allows full memory captures.

A commercial tool called F-Response allows examiners to conduct forensic acquisition remotely. F-Response use a pair of dongles, one for the suspect system and another for the forensic system.


TODO: memory dump sample and binary search
