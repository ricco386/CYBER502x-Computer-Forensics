Unit 1 - Computer Forensics Fundamentals
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Computer Forensics Concepts and Types
-------------------------------------

Computer forensic investigators use forensic tools and follow appropriate procedures to collect, preserve, analyze, and report admissible evidence to court providing his or her critical judgments of exactly what has happened.

The evidence may reside in computer systems, computer networks, computer media, computer peripherals -- basically everywhere.

Data can be in one of the three states: 

* at rest, which means stored in a computer drive, the Cloud, or a USB drive, etc, a mobile phone
* data in use, which means data is in a computer's memory currently in use
* data in transit, which means moving through a network

**System forensics** focus on evidence from volatile data, such as data in memory, and non-volatile data, which resides in hard drives; computer discs;, floppy discs; magnetic tapes; zip and the JAZ discs; log files;

**Network forensics** determines what happened on a system based on network traffic study, such as timeline analysis, IP address, or contents of the packets. This task is technically challenging since this evidence is often transient and does not last as long as stored media.

**Cloud forensics** is an emerging area focusing on Cloud-based evidence, such as Google Drives, web-based email stored on servers owned by a third party.

**Anti-digital forensics or ADF**, which are technologies designed to thwart discovery of such information. ADF approaches aim to manipulate, erase, or obfuscate digital data to make forensic examination difficult, time-consuming, or virtually impossible.

**Expert witness** present in court to judges, attorneys, juries, and other attendants to state their findings, opinions, and conclusions within the bounds of the trial.


Forensic Investigation Procedure
--------------------------------

When a computer incident is confirmed, forensic investigation starts. We should follow the company's incident response policies and procedures to decide whether turn off the suspect machine immediately or not.

Should we turn the system off?
==============================

If we turn the system off, we will lose computer memory and volatile data, such as logged-in users, PCP connections, and the running processes, etc. If possible, we should collect the volatile data before turning off the machine.

**If you turn off the system gracefully**, it ensures the **system remains in a consistent state**, since graceful shutdown includes fresh buffers to save information to disks, notify users and services, etc. **However, intruders may have installed rootkits to destroy evidence** upon receiving graceful shutdown command. You will lose volatile data, including the network state, such as network connections and arp tables, along with running processes, logged users, kernel and swap space contents.

**If you shut down the system forcefully** by yanking the power cord, it will **avoid potential loss of evidence caused by rootkits**. However, **it may cost data in cache not written to disk, and leave data in a inconsistent state**, and you will still lose volatile data.

**From a forensic perspective, you should always yank the power cord, and you have to document every action.**

The forensic procedure starts with establishing a detailed chain of custody. It is to maintain a record of how evidence has been handled, from the moment it was collected
to the moment it was present to a court. Chain-of-custody items include date and the time the evidence was collected, full name and signature of each person possessing the evidence, and for how long, location and lockers for the evidence, and whether it was stored in a tamper-proof manner. You must document all activities and transfers of the evidences from one person to another person.


Data Acquisition
----------------

With the chain of custody started, we begin the process of evidence acquisition, preservation, analysis, and reporting. Acquisition includes aquire both volatile and non-volatile data.

**Volatile data** requires power to maintain the stored information, like data in memory. Data stored on hard drives is a common example of **non-volatile data**. We always acquire volatile data first because they're short-lived.

When working on collecting evidence from a suspect machine, you have to make sure all output will be redirected outside of the suspect machine. Because otherwise you are tampering data. In a common practice, you will sanitize the evidence drive of the forensic machine before data acquisition.

A bitstream copy gets every single bit of every byte on a device. It performs on the drive level, not on a file level, ignoring the end of file marker; therefore, this process is often called hard drive imaging, bitstream imaging, or forensic imaging. Commands such as CP copy, TA, cpio, dump, restore only copy file content, stopping at the end of file marker, the bitstream copy will copy every bit on the drive, including deleted data.


Data Preservation
-----------------

Since courts require evidence be authentic and unaltered, the acquired digital evidence must be preserved in its original state. Forensics uses cryptographic hash algorithms to preserve evidence.

Cryptographic hash algorithm is a one-way function that maps data of arbitrary size, like a message, to a fixed-size bitstream or called hash value. The same message always results in the same hash value. Here one-way means it is infeasible to reverse the mapping to generate a message from its hash value. A cryptographic hash should be a collision-free algorithm. That means it is functionally impossible to find two different messages with the same hash value. So if we want to prove or authorize that two hard drive images are identical, we only need to calculate their hashes. If the hashes are same, the two images have to be the same according to the collision-free property.

`The SIFT Workstation <https://digital-forensics.sans.org/community/downloads>`_ is for incident response and digital forensics use.


Data Analysis and Report
------------------------

Whenever possible, you should protect the original physical evidence and only work with the digital copy.

We start analysis by looking at the partition table on the suspect drive to learn the number of partitions on the drive and checking for gaps between partitions for hidden data. Other analysis steps include retrieving deleted files, generating a timeline based on timestamps and the log files, finding hidden data, keyword search for terms related to your case. **Signature analysis** to identify fake extensions. **Hash analysis** to filter out both innocent files and malicious files and OS specific media analysis.

After completing the forensic examination with pertinent evidence and findings, the last step is to report your findings. As expert, you can always include your opinions in your report. For each opinion you offer, you have to provide supporting data from your analysis phase. Finally, conclude your report with your statements.
