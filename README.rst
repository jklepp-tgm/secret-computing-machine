########################
Distributed File Systems
########################

Task
====

Ori API
=======

replicate
~~~~~~~~~

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


.. header::

    +-------------+--------------------+------------+
    | ###Title### | Andreas Willinger, | 2015-03-19 |
    |             | Jakob Klepp        |            |
    +-------------+--------------------+------------+

.. footer::

    ###Page### / ###Total###
