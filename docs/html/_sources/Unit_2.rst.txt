Unit 2 - Linux/Unix Acquisition 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Acquisition Preparation and System Information Acquisition
----------------------------------------------------------

Linux/Unix forensics follows the general forensic process of **collecting, preserving, analyzing, and reporting evidence**.

1. First is a set of forensic tools, including a bootable live CD.
   
   The reason is simple: You should never trust any tools from the suspect machine since it has potentially compromised.
   When booting using a forensic bootable CD in forensic mode, the internal suspect hard drive is not touched.

2. Second is a powerful machine, sometimes we call it forensic machine or evidence server.

   This system has forensic tools installed and its drives are wiped clean for storing acquired evidence.


Volatile data acquisition
=========================

Volatile evidence is evidence that can easily be changed. Every memory dump will be different since memory is constantly changing. That's why we should first acquire the data that is most volatile.

A possible order of collection from most volatile to least volatile until non-volatile is memory; swap space or page file; network status; connections; processes running; open files; drives, and removable media.

Since you are directly collecting volatile evidence from a suspect machine, you are inevitably changing data. To ensure a minimal impact to the original data, you should always try to use small footprint tools and you should always document everything you have done.


TODO: commands sample


Memory Acquisition
------------------

For a host-based memory dump approach, the investigator needs to have physical access to the system. MEMDUMP is a part of the coroner's toolkit, TCT. To overcome memdump limitations, open source tools, Linux Memory Extractor, LiME, and Fmem were developed. Both tools will load a kernel module to the system that allows full memory captures.

A commercial tool called F-Response allows examiners to conduct forensic acquisition remotely. F-Response use a pair of dongles, one for the suspect system and another for the forensic system.


TODO: memory dump sample and binary search