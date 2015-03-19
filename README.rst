########################
Distributed File Systems
########################

Task
====

Installation und Implementierung
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

"Ori is a distributed file system built for offline operation and empowers the
user with control over synchronization operations and conflict resolution. We
provide history through light weight snapshots and allow users to verify the
history has not been tampered with. Through the use of replication instances
can be resilient and recover damaged data from other nodes." [1]_

Installieren Sie Ori und testen Sie die oben beschriebenen Eckpunkte dieses
verteilten Dateisystems (DFS). Verwenden Sie dabei auf jeden Fall alle
Funktionalitäten der API von Ori um die Einsatzmöglichkeiten auszuschöpfen.
Halten Sie sich dabei zuallererst an die Beispiele aus dem Paper im Kapitel 2
[3]_.  Zeigen Sie mögliche Einsatzgebiete für Backups und Roadwarriors (z.B.
Laptopbenutzer möchte Daten mit zwei oder mehreren Servern synchronisieren).
Führen Sie auch die mitgelieferten Tests aus und kontrollieren Sie deren
Ausgaben (Hilfestellung durch Wiki [2]_).

Gegenüberstellung
~~~~~~~~~~~~~~~~~

Wo gibt es Überschneidungen zu anderen Implementierungen von DFS? Listen Sie
diese auf und dokumentieren Sie mögliche Entscheidungsgrundlagen für mindestens
zwei unterschiedliche Einsatzgebiete. Verwenden Sie dabei zumindest HDFS [4]_
und GlusterFS [5]_ als Gegenspieler zu Ori. Weitere Implementierungen sind
möglich aber nicht verpflichtend. Um aussagekräftige Vergleiche anstellen zu
können, wäre es von Vorteil die anderen Systeme ebenfalls - zumindest
oberflächlich - zu testen.

Info
~~~~

Gruppengröße: 2 Mitglieder
Gesamtpunkte: 16

* Installation und Testdurchlauf von Ori: 2 Punkte
* Einsatz/Dokumentation der Ori API (replicate, snapshot, checkout, graft,
  filelog, list, log, merge, newfs, pull, remote, removefs, show, status,
  tip, varlink): 8 Punkte
* Gegenüberstellungstabelle: 4 Punkte
* Einsatz der Gegenspieler: 2 Punkte


Ori API
=======

replicate
~~~~~~~~~

The replicate operation creates a new replica of a file
system when given a path to a source replica (local or
remote). It works by first creating and configuring an
empty repository. Next, it retrieves the hash of latest
commit from the source and saves it for later use. It
then scans the source for all relevant objects and transfers
them to the destination. The set of relevant objects
depends on whether the new instance is a full replica
(including history) or shallow replica. Finally, when the
transfer is complete, the new file system head is set to
point to the latest commit object that was transfered. [3]_

snapshot
~~~~~~~~

checkout
~~~~~~~~

graft
~~~~~

filelog
~~~~~~~

list
~~~~

log
~~~

merge
~~~~~

newfs
~~~~~

pull
~~~~

Pull updates an existing repository and transfers only missing data, but does
not merge new changes into a mounted file system. (Note this operation is
closer to git fetch than git pull.) [3]_

remote
~~~~~~

removefs
~~~~~~~~

show
~~~~

status
~~~~~~

tip
~~~

varlink
~~~~~~~

Troubles
========

Time out of sync
~~~~~~~~~~~~~~~~

Both host have to have the same time.
To archive that the ntpd daemon should run.

On a system with systemd ntpd can be enabled with this command:

.. code:: bash

    sudo systemctl enable ntpd

The *STABLE* version does not work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A new version called ori-orisyncng fixes some of the problems.
It is available in the following git repo:

.. code:: text

    https://bitbucket.org/orifs/ori-orisyncng.git

BUFFER
======

[jakob@manj 2014-2015]$ orisync init
Is this the first machine in the cluster (y/n)? n
Enter the cluster name: DezSys
Enter the cluster key: t8jhfhkm

Use the following configuration for all other machines:
Cluster Name: DezSys
Cluster Key:  t8jhfhkm

Now use 'orisync add' to register repositories.
[jakob@manj 2014-2015]$ ori replicate schueler@192.168.1.26:MyRepo
Cloning from schueler@192.168.1.26:MyRepo to /home/jakob/.ori/MyRepo.ori
Enter passphrase for key '/home/jakob/.ssh/id_rsa': 
[jakob@manj 2014-2015]$ 
[jakob@manj 2014-2015]$ orisync add /home/jakob/.ori/MyRepo.ori
[jakob@manj 2014-2015]$ orisync
OriSync started as pid 1465
[jakob@manj 2014-2015]$ orisync list
Repo                            Mounted                         Peers                           
/home/jakob/.ori/MyRepo.ori     false                           192.168.1.26                    
[jakob@manj 2014-2015]$ mkdir MyRepo
[jakob@manj 2014-2015]$ orifs MyRepo
[jakob@manj 2014-2015]$ ori list
Name                            File System ID
MyRepo                          d252249a-25e6-464d-a0e8-aa16aac85bfe
[jakob@manj 2014-2015]$ cd MyRepo/
[jakob@manj MyRepo]$ ls
MYDIR/  MYFILE
[jakob@manj MyRepo]$ ls
MYDIR/  MYFILE
[jakob@manj MyRepo]$ less MYFILE 
[jakob@manj MyRepo]$ less MYFILE 
[jakob@manj MyRepo]$ cat ~/.ori/
MyRepo.ori/  orisync.log  orisyncrc    orisyncSock  
[jakob@manj MyRepo]$ cat ~/.ori/orisync.log 
[jakob@manj MyRepo]$ cat ~/.ori/orisync.log 
[jakob@manj MyRepo]$ cat ~/.ori/orisync.log 
[jakob@manj MyRepo]$ ls
MYDIR/  MYFILE
[jakob@manj MyRepo]$ ls -la
total 7
drwxr-xr-x  3 jakob users  512 19.03.2015 13:14 ./
drwxr-xr-x 20 jakob users 4096 19.03.2015 13:11 ../
drwxr-xr-x  2 jakob users  512 19.03.2015 13:13 MYDIR/
drwxr-xr-x  2 jakob users  512 01.01.1970 01:00 .snapshot/
-rw-r--r--  1 jakob users    3 19.03.2015 13:13 MYFILE
-rw-------  1 jakob users   27 01.01.1970 01:00 .ori_control
[jakob@manj MyRepo]$ cat ~/.ori/
MyRepo.ori/  orisync.log  orisyncrc    orisyncSock  
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/
HEAD       id         index      metadata   objs/      ori.log    README     refs/      snapshots  tmp/       trusted/   uds        version    
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/o
objs/    ori.log  
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/ori.log 
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/
HEAD       id         index      metadata   objs/      ori.log    README     refs/      snapshots  tmp/       trusted/   uds        version    
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/
HEAD       id         index      metadata   objs/      ori.log    README     refs/      snapshots  tmp/       trusted/   uds        version    
[jakob@manj MyRepo]$ cat ~/.ori/MyRepo.ori/snapshots 
c50e8a29d0cb810fc33b6b1b386a244bc33e540d5d6bb91a7f057cc8e2bc4567 1426765932 Orisync snapshot
adbe6a8d9ca0e715fd59f11faea7b37eae867484b73fdca6259b1a0b145c3ddb 1426767209 Orisync snapshot
90080b02f225ff8357a10a2ae9b46d0fbc2059faa8cdb9f8850ebdeed58d78c1 1426767212 Orisync snapshot
26139a63e1b563df7ad1f7e566f4acb42dac9cf227e682945623fc867a83b15a 1426767246 Orisync snapshot
[jakob@manj MyRepo]$ 
Display all 5453 possibilities? (y or n)
[jakob@manj MyRepo]$ cd ..
[jakob@manj 2014-2015]$ ls
AM/   FasterThanPortfolio/  MyRepo/  VSDB/               zugerkennung-repo2/                                 FT-Portfolio.odt
BIM/  IndInf/               NWTK/    WIR/                10614189_810418922336761_7922793367701312512_n.jpg  stundenplan.png
D/    INSY/                 SEW/     Zugerkennung/       2014-07-03_Zugmessung_mono_min3545to3630.wav        systemtechnik_schuelereinteilung_20141105_1100.xls
E/    ITP/                  SYT/     zugerkennung-repo/  Frantar_Elias_S22.wxmx
[jakob@manj 2014-2015]$ cd MyRepo/
[jakob@manj MyRepo]$ ori 
Ori Distributed Personal File System (Version 0.8.2) - Command Line Interface

Available commands:
filelog         Display a log of change to the specified file
help            Show help for a given topic
list            List local file systems
log             Display a log of commits to the repository
merge           Merge two heads
newfs           Create a new file system
pull            Pull changes from a repository
remote          Remote connection management
removefs        Remove a local replica
replicate       Create a local replica
show            Show repository information
snapshot        Create a repository snapshot
snapshots       List all snapshots available in the repository
status          Scan for changes since last commit
tip             Print the latest commit on this branch
varlink         Get, set, list varlink variables

Please report bugs to orifs-devel@stanford.edu
Website: http://ori.scs.stanford.edu/
[jakob@manj MyRepo]$ ori filelog 
MYDIR/        MYFILE        .ori_control  .snapshot/    
[jakob@manj MyRepo]$ ori filelog 
MYDIR/        MYFILE        .ori_control  .snapshot/    
[jakob@manj MyRepo]$ ori filelog MYFILE 
Commit:  26139a63e1b563df7ad1f7e566f4acb42dac9cf227e682945623fc867a83b15a
Parents: 90080b02f225ff8357a10a2ae9b46d0fbc2059faa8cdb9f8850ebdeed58d78c1
Author:  Schueler,,,
Date:    Thu Mar 19 13:14:06 2015

Orisync automatic snapshot

Commit:  90080b02f225ff8357a10a2ae9b46d0fbc2059faa8cdb9f8850ebdeed58d78c1
Parents: adbe6a8d9ca0e715fd59f11faea7b37eae867484b73fdca6259b1a0b145c3ddb
Author:  Schueler,,,
Date:    Thu Mar 19 13:13:32 2015

Orisync automatic snapshot

[jakob@manj MyRepo]$ ori list
Name                            File System ID
MyRepo                          d252249a-25e6-464d-a0e8-aa16aac85bfe
[jakob@manj MyRepo]$ ori log
Commit:    26139a63e1b563df7ad1f7e566f4acb42dac9cf227e682945623fc867a83b15a
Parents:   90080b02f225ff8357a10a2ae9b46d0fbc2059faa8cdb9f8850ebdeed58d78c1 
Tree:      d38a7f034823cf49bd429d29919de66e0491751cc1ca5493e350c001be62a7ff
Author:    Schueler,,,
Date:      Thu Mar 19 13:14:06 2015

Orisync automatic snapshot

Commit:    90080b02f225ff8357a10a2ae9b46d0fbc2059faa8cdb9f8850ebdeed58d78c1
Parents:   adbe6a8d9ca0e715fd59f11faea7b37eae867484b73fdca6259b1a0b145c3ddb 
Tree:      882af59222f436f9c5b3d5a948987cf1b594ff6f8906dcba3251d647656fc20d
Author:    Schueler,,,
Date:      Thu Mar 19 13:13:32 2015

Orisync automatic snapshot

Commit:    adbe6a8d9ca0e715fd59f11faea7b37eae867484b73fdca6259b1a0b145c3ddb
Parents:   c50e8a29d0cb810fc33b6b1b386a244bc33e540d5d6bb91a7f057cc8e2bc4567 
Tree:      32c549107a55199b5ef08eb8a50b900f657b8b9e64da44735da8b59cbbe74aea
Author:    Schueler,,,
Date:      Thu Mar 19 13:13:29 2015

Orisync automatic snapshot

Commit:    c50e8a29d0cb810fc33b6b1b386a244bc33e540d5d6bb91a7f057cc8e2bc4567
Parents:    
Tree:      aca9e87acc0602718b46f7b4dd5edc6b6d678d8f3246a252b5017cb794e9d92f
Author:    Schueler,,,
Date:      Thu Mar 19 12:52:12 2015

Orisync automatic snapshot

[jakob@manj MyRepo]$ ori remote
Name            Path                                                            
ID
origin          schueler@192.168.1.26:MyRepo                                    

[jakob@manj MyRepo]$ ori help remote
No help for command 'remote'
[jakob@manj MyRepo]$ ori show
--- Repository ---
Root: /home/jakob/.ori/MyRepo.ori
UUID: d252249a-25e6-464d-a0e8-aa16aac85bfe
Version: ORI1.1
HEAD: 26139a63e1b563df7ad1f7e566f4acb42dac9cf227e682945623fc867a83b15a
[jakob@manj MyRepo]$ ori status
[jakob@manj MyRepo]$ ori tip
0efbfaee847b85fb4d44462f2e2dcb5de8610cf8468c8786a9147776773eb1be
[jakob@manj MyRepo]$ ori
Ori Distributed Personal File System (Version 0.8.2) - Command Line Interface

Available commands:
filelog         Display a log of change to the specified file
help            Show help for a given topic
list            List local file systems
log             Display a log of commits to the repository
merge           Merge two heads
newfs           Create a new file system
pull            Pull changes from a repository
remote          Remote connection management
removefs        Remove a local replica
replicate       Create a local replica
show            Show repository information
snapshot        Create a repository snapshot
snapshots       List all snapshots available in the repository
status          Scan for changes since last commit
tip             Print the latest commit on this branch
varlink         Get, set, list varlink variables

Please report bugs to orifs-devel@stanford.edu
Website: http://ori.scs.stanford.edu/
[jakob@manj MyRepo]$ ori varlink
Variable        Value                                                           
machtype        unknown                                                         
osname          unknown                                                         
domainname      (none)                                                          
hostname        manj                                                            
[jakob@manj MyRepo]$ ori varlink machtype=1

[jakob@manj MyRepo]$ ori varlink
Variable        Value                                                           
machtype        unknown                                                         
osname          unknown                                                         
domainname      (none)                                                          
hostname        manj                                                            
[jakob@manj MyRepo]$ ori varlink machtype 1
[jakob@manj MyRepo]$ ori varlink
Variable        Value                                                           
machtype        unknown                                                         
osname          unknown                                                         
domainname      (none)                                                          
hostname        manj                                                            
machtype        unknown                                                         
[jakob@manj MyRepo]$ ori varlink machtype
unknown
[jakob@manj MyRepo]$ ori varlink
Variable        Value                                                           
machtype        unknown                                                         
osname          unknown                                                         
domainname      (none)                                                          
hostname        manj                                                            
machtype        unknown                                                         
[jakob@manj MyRepo]$ ori varlink set machtype BLUB
Wrong number of arguments!
Usage: ori varlist - List variables
Usage: ori varlist VARIABLE - Get variable
Usage: ori varlist VARIABLE VALUE - Set variable
[jakob@manj MyRepo]$ ori varlink machtype BLUB
[jakob@manj MyRepo]$ ori varlink
Variable        Value                                                           
machtype        unknown                                                         
osname          unknown                                                         
domainname      (none)                                                          
hostname        manj                                                            
machtype        unknown        


Comparison
==========

Hadoop Distributed File System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

HDFS is the primary distributed storage used by Hadoop applications. A HDFS
cluster primarily consists of a NameNode that manages the file system metadata
and DataNodes that store the actual data. The HDFS Architecture Guide describes
HDFS in detail. This user guide primarily deals with the interaction of users
and administrators with HDFS clusters. The HDFS architecture diagram depicts
basic interactions among NameNode, the DataNodes, and the clients. Clients
contact NameNode for file metadata or file modifications and perform actual
file I/O directly with the DataNodes. [4]_

GlusterFS
~~~~~~~~~

GlusterFS is an open source, distributed file system capable of scaling to
several petabytes (actually, 72 brontobytes!) and handling thousands of
clients. GlusterFS clusters together storage building blocks over Infiniband
RDMA or TCP/IP interconnect, aggregating disk and memory resources and
managing data in a single global namespace. GlusterFS is based on a stackable
user space design and can deliver exceptional performance for diverse
workloads. [5]_

Table
~~~~~

======================== ===================== ===================== ========================
 -                       **Ori File System**   **HDFS**              **GlusterFS**
======================== ===================== ===================== ========================
**Supported OS**         Linux, FreeBSD,       Linux,                Linux,
                         Mac OS X              some other Unix,      Mac OS X,
                         ,                     Java API on other     Windows
                                               OS too
**Use case**             Replication; in       Big data;             network-attached storage
                         future bidirectional  write-one-read-many
                         synchronization
**Architecture**         Peer to Peer          master/slave [7]_     master/slave
**Replication Strategy** polling (every 5      Replicates created at
                         seconds)              the time of writing
                                               [6]_
**Security**             Transmission via SSH; None [6]_             None, don't expose the
                                                                     server nodes [9]_
**Consistency**          merkle tree
======================== ===================== ===================== ========================

Sources
=======

.. _1:

[1] Ori File System, Stanford Website,
    online: http://ori.scs.stanford.edu/,
    visited: 2015-03-02

.. _2:

[2] Ori File System, Bitbucket Wiki,
    online: https://bitbucket.org/orifs/ori/wiki/Home,
    visited: 2015-03-02

.. _3:

[3] Ali José Mashtizadeh, Andrea Bittau, Yifeng Frang Huang, David Mazières.
    Replication, History, and Grafting in the Ori File System.
    In Proceedings of the 24th Symposium on Operating Systems Principles,
    November 2013. Paper.

.. _4:

[4] Apache Hadoop FileSystem,
    http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html,
    visited: 2015-03-19

.. _5:

[5] GlusterFS,
    http://www.gluster.org/documentation/howto/HowTo/,
    visited: 2015-03-19

.. _6:

[6] Large Scale Distributed File System Survey, Yuduo Zhou,
    http://grids.ucs.indiana.edu/ptliupages/publications/Large%20Scale%20Distributed%20File%20System%20Survey.pdf,
    visited: 2015-03-19

.. _7:

[7] Distributed File Systems: A Survey, L.Sudha Rani, K. Sudhakar, S.Vinay Kumar,
    http://www.ijcsit.com/docs/Volume%205/vol5issue03/ijcsit20140503234.pdf,
    visited: 2015-03-19

.. _9:

[9] Security concerns with glusterfs?
    http://serverfault.com/questions/659677/security-concerns-with-glusterfs,
    visited: 2015-03-19

.. header::

    +-------------+--------------------+------------+
    | ###Title### | Andreas Willinger, | 2015-03-19 |
    |             | Jakob Klepp        |            |
    +-------------+--------------------+------------+

.. footer::

    ###Page### / ###Total###
