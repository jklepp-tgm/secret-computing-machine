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

Installation
============

We first tried to get the latest stable version (0.8.1) working, but orisync did
not work properly.

For more information, see the troubles section at the end.

After a forum post made by our teacher, Michael Borko, we built Ori
from the Bitbucket repository directly/a developer build.

Orisync worked after building ori by executing the following steps:

.. code:: bash

    sudo apt-get install scons build-essential pkg-config
    sudo apt-get install libboost-dev uuid-dev libfuse-dev libevent-dev libssl-dev
    sudo apt-get install libedit-dev
    cd downloads/
    git clone https://bitbucket.org/orifs/ori-orisyncng.git
    cd ori-orisyncng/
    # Build Ori
    scons
    # Change the installation path prefix
    vim SConstruct
    # Replace the value of PREFIX with /usr/local/
    # Install Ori
    sudo scons install
    # Then copy over the SSH public key into the authorized hosts file
    cd ~/.ssh
    cat id_rsa.pub >> authorized_keys

With that, the installation is complete.

Running the tests
=================

First, some important notes from the official Bitbucket repository:

To run the test suite you will need to configure an SSH public key to access
your local machine without a password. You will also want to save the following
into runtests_config.sh.

.. code:: bash

    # Required for Mac OS X and FreeBSD only (comment out on Linux machines)
    # We commented this out due to running it on Debian
    #export UMOUNT="umount"

    # Not updated to new CLI
    HTTP_CLONE="no"
    HTTP_PULL="no"
    MERGE="no"
    MOUNT_WRITE="no"
    MOUNT_WRITE_PYTHON_MT="no"

Once configured you can run runtests.sh. On an error you may have to cleanup the
tempdir and test repositories on your system before rerunning. The logs will be
available inside the tempdir if an error occurred. [8]_

This was the output of executing runtests.sh on our machine:

.. code:: text

    runtests.sh test results
    Fri Mar  6 09:04:11 CET 2015

    01-new-remove-fs
    02-local-clone
    03-local-clone-empty
    04-local-pull
    05-local-clone-uds
    11-ssh-clone
    12-ssh-clone-empty
    14-ssh-pull
    15-ssh-clone-uds
    SKIPPED: 21-http-clone
    SKIPPED: 22-http-pull
    30-local-clone-shallow
    SKIPPED: 35-merge
    SKIPPED: 51-mount-write
    52-mount-write-zlib
    53-mount-write-wget
    60-mount-write-zlib-mt
    61-mount-write-wget-mt
    SKIPPED: 62-mount-write-python-mt

Ori API
=======

*Important*

All commands (except orisync itself and pull/merge/checkout/remote)
have been done after setting up orisync and the outputs of them are therefore
not manually done.

orisync
~~~~~~~

For synchronization to run properly, one first has to initialize a repo on one
side and then replicate it to the other.

During our tests it turned out that orisync only supports unidirectional
synchronization.

This means that changes on the "slave" will not be copied over to the "master".

The following commands have been used in this section:

newfs, replicate, list and orisync itself.

Master initialization
---------------------

The master repository is hosted on Willinger's Debian 8 testing VM.

.. code:: bash

    # Initialize orisync
    # As this host is the master, answer with yes
    schueler@debian:~/DezSys$ orisync init
    Is this the first machine in the cluster (y/n)? y
    Enter the cluster name: DezSys

    Use the following configuration for all other machines:
    Cluster Name: DezSys
    Cluster Key:  t8jhfhkm

    Now use 'orisync add' to register repositories.
    # Create a new repository
    # NOTE: THIS DOES NOT MOUNT ANYTHING
    # Mounting still has to be done later by using orifs
    schueler@debian:~/DezSys$ ori newfs MyRepo
    # Add the just created repository to orisync, so it is actually going to be
    # synchronized.
    schueler@debian:~/DezSys$ orisync add /home/schueler/.ori/MyRepo.ori
    # Start orisync
    # It now looks for changes in the background, each 5 seconds
    schueler@debian:~/DezSys$ orisync
    OriSync started as pid 59072
    # Verification
    schueler@debian:~/DezSys$ orisync list
    Repo                            Mounted                         Peers                           
    /home/schueler/.ori/MyRepo.ori  false                                                           
    # Setup a mount point
    schueler@debian:~/DezSys$ mkdir MyRepo
    # Mount the repo
    # Now we can just use it like a normal direcotry
    schueler@debian:~/DezSys$ orifs MyRepo
    # Verify that it's actually mounted
    schueler@debian:~/DezSys$ mount
    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
    proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
    udev on /dev type devtmpfs (rw,relatime,size=10240k,nr_inodes=255226,mode=755)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
    tmpfs on /run type tmpfs (rw,nosuid,relatime,size=410876k,mode=755)
    /dev/sda2 on / type ext4 (rw,relatime,errors=remount-ro,user_xattr,barrier=1,data=ordered)
    securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
    tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
    tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
    tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
    cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
    cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
    cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
    cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
    cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
    cgroup on /sys/fs/cgroup/net_cls type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls)
    cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
    cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
    systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=22,pgrp=1,timeout=300,minproto=5,maxproto=5,direct)
    mqueue on /dev/mqueue type mqueue (rw,relatime)
    hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
    debugfs on /sys/kernel/debug type debugfs (rw,relatime)
    fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
    /dev/sda1 on /boot type ext2 (rw,relatime,errors=continue)
    vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other)
    rpc_pipefs on /run/rpc_pipefs type rpc_pipefs (rw,relatime)
    tmpfs on /run/user/111 type tmpfs (rw,nosuid,nodev,relatime,size=205440k,mode=700,uid=111,gid=118)
    binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
    tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=205440k,mode=700,uid=1000,gid=1000)
    orifs on /home/schueler/DezSys/MyRepo type fuse.orifs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000) # <--
    # Now orisync also shows "mounted"
    schueler@debian:~/DezSys$ orisync list
    Repo                            Mounted                         Peers                           
    /home/schueler/.ori/MyRepo.ori  true                                                            

Slave initialization
--------------------

These steps have been done on Jakob's laptop running Manjaro.

.. code:: bash

    # Add this host to orisync
    [jakob@manj 2014-2015]$ orisync init
    Is this the first machine in the cluster (y/n)? n
    Enter the cluster name: DezSys
    Enter the cluster key: t8jhfhkm

    Use the following configuration for all other machines:
    Cluster Name: DezSys
    Cluster Key:  t8jhfhkm

    Now use 'orisync add' to register repositories.
    # Replicate the repository from the master
    [jakob@manj 2014-2015]$ ori replicate schueler@192.168.1.26:MyRepo
    Cloning from schueler@192.168.1.26:MyRepo to /home/jakob/.ori/MyRepo.ori
    Enter passphrase for key '/home/jakob/.ssh/id_rsa': 
    # Also add this repository to orisync, synchronize it
    [jakob@manj 2014-2015]$ orisync add /home/jakob/.ori/MyRepo.ori
    # Start orisync
    [jakob@manj 2014-2015]$ orisync
    OriSync started as pid 1465
    # Verification
    # If there is no value at "Peers" then something went really wrong
    [jakob@manj 2014-2015]$ orisync list
    Repo                            Mounted                         Peers                           
    /home/jakob/.ori/MyRepo.ori     false                           192.168.1.26                   
    # And mount the repository here too
    [jakob@manj 2014-2015]$ mkdir MyRepo
    [jakob@manj 2014-2015]$ orifs MyRepo
    # Verification
    # Lists all repositores available in /home/$USER/.ori/
    [jakob@manj 2014-2015]$ ori list
    Name                            File System ID
    MyRepo                          d252249a-25e6-464d-a0e8-aa16aac85bfe

In the following we have done some testing if the synchronization works.

*Master*

.. code:: bash

    schueler@debian:~/DezSys$ l
    total 1
    drwxr-xr-x 2 schueler schueler 512 Mar 19 12:52 MyRepo
    schueler@debian:~/DezSys$ cd MyRepo/
    schueler@debian:~/DezSys/MyRepo$ l
    total 1
    -rw------- 1 schueler schueler  30 Jan  1  1970 .ori_control
    drwxr-xr-x 2 schueler schueler 512 Jan  1  1970 .snapshot
    schueler@debian:~/DezSys/MyRepo$ mkdir MYDIR
    schueler@debian:~/DezSys/MyRepo$ touch MYFILE
    schueler@debian:~/DezSys/MyRepo$ echo "HI" > MYFILE
    schueler@debian:~/DezSys/MyRepo$ cat MYFILE
    HI
    schueler@debian:~/DezSys/MyRepo$

Please note that orisync only looks for changes each 5 seconds.

.. code:: bash

    [jakob@manj 2014-2015]$ cd MyRepo/
    [jakob@manj MyRepo]$ ls
    MYDIR/  MYFILE
    # This has been done before doing echo "HI" > MYFILE on the master
    [jakob@manj MyRepo]$ less MYFILE
    # And this afterwards
    [jakob@manj MyRepo]$ cat MYFILE
    HI

Orisync now watches, as mentioned, every 5 seconds for changes and performs a 
pull on the client each time something has changed.

Orisync is useful for people, who need to have the same files on multiple locations
(like on your home computer and laptop) and do not want to use a external service.


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

Create a repository snapshot, acts like a commit in git.

Example:

.. code:: bash

    # Create a new repository for testing
    schueler@debian:~/DezSys$ ori newfs NoRepo
    # Mount it
    schueler@debian:~/DezSys$ mkdir NoRepo
    schueler@debian:~/DezSys$ orifs NoRepo
    # New repo, so nothing has happened yet
    schueler@debian:~/DezSys/NoRepo$ ori status
    schueler@debian:~/DezSys/NoRepo$ mkdir TEST
    # Nothing has changed since the last snapshot, as we don't even have one
    # yet.
    schueler@debian:~/DezSys/NoRepo$ ori status
    # Create a first snapshot
    schueler@debian:~/DezSys/NoRepo$ ori snapshot FIRST
    Committed b98e0ed99b83c2d30f68541e629922405436a3131ece08b5beea71315187011d
    # Now the changes have been comitted
    schueler@debian:~/DezSys/NoRepo$ ori status
    # Create a new file
    schueler@debian:~/DezSys/NoRepo$ touch MYFILE
    # And we see that something has changed since the last snapshot
    schueler@debian:~/DezSys/NoRepo$ ori status
    A   /MYFILE
    # Create a second snapshot
    schueler@debian:~/DezSys/NoRepo$ ori snapshot SECOND
    Committed 714b922251dc9efc00acd9fa614dbf68995a6b9947e3bd0f1a57f24a9eebcc33
    # On the latest status
    schueler@debian:~/DezSys/NoRepo$ ori status
    schueler@debian:~/DezSys/NoRepo$ 

This command is useful for backing up data.

All snapshots are saved in the repository under the .snapshots/SNAPSHOTNAME
folder and can be accssed just like normal directories.

Therefore, file recoveries are easily done (as long as your hard disk did not
crash).

Example for recovery:

.. code:: bash

    # Delete a file
    schueler@debian:~/DezSys/NoRepo$ rm MYFILE
    # Ori has noticed that the file was deleted since the last snapshot
    schueler@debian:~/DezSys/NoRepo$ ori status
    D   /MYFILE
    # Restore the file
    schueler@debian:~/DezSys/NoRepo$ cp .snapshot/SECOND/MYFILE .
    # It's no longer deleted, but modifed
    schueler@debian:~/DezSys/NoRepo$ ori status
    M   /MYFILE

checkout
~~~~~~~~

Similar to git checkout.
Sets the current HEAD to the specified snapshot.

Mostly done after a successful pull.

Example:

.. code:: bash

    [jakob@manj MyRepo2]$ ls
    [jakob@manj MyRepo2]$
    # Pull here
    [jakob@manj MyRepo2]$ ori checkout
    714b922251dc9efc00acd9fa614dbf68995a6b9947e3bd0f1a57f24a9eebcc33
    Checkout success!
    [jakob@manj MyRepo2]$ ls
    TEST/  MYFILE
    [jakob@manj MyRepo2]$ 

graft
~~~~~

"
[..]
Using a novel feature called grafts, one can copy a subtree of one file system
to another file system in such a way as to preserve the file history and
relationship of the two directories.
Grafts can be explicitly re-synchronized in either direction, providing a facility
similar to a distributed version control system (DVCS) such as Git.
However, with one big difference: in a DVCS, one must decide ahead of time that a
particular directory will be a repository; while in Ori, any directory can be
grafted at any time.

By grafting instead of copying, one can later determine whether one copy of a
file contains all changes in another (a common question when files have been copied
across file systems and edited in multiple places).
" [3]_

**Example 1**

FS 1 -> FS 2

.. code:: bash

    schueler@debian:~/DezSys$ cd GraftA/
    schueler@debian:~/DezSys/GraftA$ mkdir A_DIR
    schueler@debian:~/DezSys/GraftA$ ls
    A_DIR
    schueler@debian:~/DezSys/GraftA$ cd ..
    schueler@debian:~/DezSys$ cd GraftB
    schueler@debian:~/DezSys/GraftB$ ls
    schueler@debian:~/DezSys/GraftB$ ori graft ~/DezSys/GraftA/A_DIR/ ~/DezSys/GraftB
    Warning: source or destination is not a repository.
    schueler@debian:~/DezSys/GraftB$ ls
    A_DIR
    schueler@debian:~/DezSys/GraftB$ 

The warning can be ignored, the command still worked.

**Example 2**

FS 2 -> FS1

.. code:: bash

    schueler@debian:~/DezSys/GraftB$ ls
    A_DIR
    schueler@debian:~/DezSys/GraftB$ mkdir B_DIR
    schueler@debian:~/DezSys/GraftB$ cd ..
    schueler@debian:~/DezSys$ cd GraftA
    schueler@debian:~/DezSys/GraftA$ ls
    A_DIR
    schueler@debian:~/DezSys/GraftA$ ori graft ~/DezSys/GraftB/B_DIR/ ~/DezSys/GraftA
    Warning: source or destination is not a repository.
    schueler@debian:~/DezSys/GraftA$ ls
    A_DIR  B_DIR
    schueler@debian:~/DezSys/GraftA$ 

filelog
~~~~~~~

Display a log of change to the specified file.

Example:

.. code:: bash

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

Here there are two commits, one for creating the file and one for changing it.

list
~~~~

List local file system - or repositories.

Example: 

.. code:: bash

    [jakob@manj MyRepo]$ ori list
    Name                            File System ID
    MyRepo                          d252249a-25e6-464d-a0e8-aa16aac85bfe

log
~~~

Display a log of commits to the repository.

Example:

.. code:: bash

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


merge
~~~~~

During our tests we found out that doing a checkout to the target commitid is
mandatory in order for the merge function to work.

If you don't do a checkout beforehand, the following error occurs:

.. code:: bash

    [jakob@manj MyRepo2]$ ori merge 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    merge failed with an unknown error!
    [jakob@manj MyRepo2]$ ls
    ls: cannot open directory .: Transport endpoint is not connected
    [jakob@manj MyRepo2]$ ls
    ls: cannot open directory .: Transport endpoint is not connected 

Essentially breaking your entire repository.

When using a checkout beforehand, the merge succeeds.

.. code:: bash

    [jakob@manj MyRepo2]$ ls
    TEST/ MYFILE
    # Pull here
    [jakob@manj MyRepo2]$ ori checkout 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    Checkout success!
    [jakob@manj MyRepo2]$ ori merge 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    Merge success!
    [jakob@manj MyRepo2]$ ls
    TEST/ KLEPP MYFILE 
    [jakob@manj MyRepo2]$

pull
~~~~

Pull changes from a repository.

Pull updates an existing repository and transfers only missing data, but does
not merge new changes into a mounted file system. (Note this operation is
closer to git fetch than git pull.) [3]_

This command can be used in two ways:

**First**

Pulling using a full address without a ori remote add beforehand.

.. code:: bash

    [jakob@manj MyRepo2]$ ori pull schueler@192.168.1.26:NoRepo
    Pulled up to 714b922251dc9efc00acd9fa614dbf68995a6b9947e3bd0f1a57f24a9eebcc33
    [jakob@manj MyRepo2]$

**Second**

After adding a remote origin using remote add, the URL paramter can be left out.

This only pulls changes "commited" by using the snapshot function.

.. code:: bash

    [jakob@manj MyRepo2]$ ori pull
    Pulled up to 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    [jakob@manj MyRepo2]$

Now, we make a change on the master repo:

.. code:: bash

    schueler@debian:~/DezSys/NoRepo$ touch KLEPP
    schueler@debian:~/DezSys/NoRepo$ ori snapshot THIRD
    Committed 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    schueler@debian:~/DezSys/NoRepo$

And pull it on the slave:

.. code:: bash

    [jakob@manj MyRepo2]$ ori pull
    Pulled up to 9377ef5daf032eaeb66b73c50a4e4bdcca3b4b052a7c836b05dd4affd0a54708
    [jakob@manj MyRepo2]$

remote
~~~~~~

Remote connection management. Also similar to git's remote command.

Example output when using orisync:

.. code:: bash

    [jakob@manj MyRepo]$ ori remote
    Name            Path                                                            
    ID
    origin          schueler@192.168.1.26:MyRepo

Or, when using ori pull, one can also define an origin to pull from, so it is
not needed to enter the full address every time.

Example:

.. code:: bash

    # The following three steps can be left out when working on a existing repo
    [jakob@manj 2014-2015]$ ori newfs MyRepo2
    [jakob@manj 2014-2015]$ mkdir MyRepo2
    [jakob@manj 2014-2015]$ orifs MyRepo2
    [jakob@manj 2014-2015]$ cd MyRepo2
    [jakob@manj 2014-2015]$ cd MyRepo2/
    [jakob@manj MyRepo2]$ ls
    [jakob@manj MyRepo2]$ ori remote add origin schueler@192.168.1.26:NoRepo
    [jakob@manj MyRepo2]$

Now it is possible to use git pull without specifying the full URL.

removefs
~~~~~~~~

Remove a local repository.

Example:

.. code:: bash

    schueler@debian:~/DezSys$ ori removefs NoRepo
    schueler@debian:~/DezSys$

IMPORTANT! This does not automatically unmount the repository/filesystem and can
lead to strange behaviour when writing to the mount point.

Proof it is still mounted:

.. code:: bash

    schueler@debian:~/DezSys$ mount
    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
    proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
    udev on /dev type devtmpfs (rw,relatime,size=10240k,nr_inodes=255226,mode=755)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
    tmpfs on /run type tmpfs (rw,nosuid,relatime,size=410876k,mode=755)
    /dev/sda2 on / type ext4 (rw,relatime,errors=remount-ro,user_xattr,barrier=1,data=ordered)
    securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
    tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
    tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
    tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
    cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
    cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
    cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
    cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
    cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
    cgroup on /sys/fs/cgroup/net_cls type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls)
    cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
    cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
    systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=22,pgrp=1,timeout=300,minproto=5,maxproto=5,direct)
    mqueue on /dev/mqueue type mqueue (rw,relatime)
    hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
    debugfs on /sys/kernel/debug type debugfs (rw,relatime)
    fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
    /dev/sda1 on /boot type ext2 (rw,relatime,errors=continue)
    vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other)
    rpc_pipefs on /run/rpc_pipefs type rpc_pipefs (rw,relatime)
    tmpfs on /run/user/111 type tmpfs (rw,nosuid,nodev,relatime,size=205440k,mode=700,uid=111,gid=118)
    binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
    tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=205440k,mode=700,uid=1000,gid=1000)
    orifs on /home/schueler/DezSys/MyRepo type fuse.orifs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
    orifs on /home/schueler/DezSys/NoRepo type fuse.orifs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

Therefore, before deleting a repo, one should unmount it first.

For example by using the fusermount tool, which does not require root privileges.

Example:

.. code:: bash

    schueler@debian:~/DezSys$ fusermount -u /home/schueler/DezSys/NoRepo 
    schueler@debian:~/DezSys$

show
~~~~

Show repository information.

Example:

.. code:: bash

    [jakob@manj MyRepo]$ ori show
    --- Repository ---
    Root: /home/jakob/.ori/MyRepo.ori
    UUID: d252249a-25e6-464d-a0e8-aa16aac85bfe
    Version: ORI1.1
    HEAD: 26139a63e1b563df7ad1f7e566f4acb42dac9cf227e682945623fc867a83b15a

status
~~~~~~

Scan for changes since last commit.

When using orisync, this does not output anything, as all changes are automatically
committed and synched.

Example:

.. code:: bash

    [jakob@manj MyRepo]$ ori status
    
    [jakob@manj MyRepo]$ 

It is mainly used when doing snapshots and/or pull/merging with a remote host/
repository manually.

See the snapshot section for an example.

tip
~~~

Print the latest commit on this branch.

Example:

.. code:: bash

    [jakob@manj MyRepo]$ ori tip
    0efbfaee847b85fb4d44462f2e2dcb5de8610cf8468c8786a9147776773eb1be

varlink
~~~~~~~

Get, set, list varlink variables.

Example:

.. code:: bash

    [jakob@manj MyRepo]$ ori varlink
    Variable        Value                                                           
    machtype        unknown                                                         
    osname          unknown                                                         
    domainname      (none)                                                          
    hostname        manj                                                            
    [jakob@manj MyRepo]$

Troubles
========

Time out of sync
~~~~~~~~~~~~~~~~

Both hosts have to have the same time.
To archive that the ntpd daemon should run.

On a system with systemd ntpd can be enabled with this command:

.. code:: bash

    sudo systemctl enable ntpd

The *STABLE* version does not work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As mentioned in the installation section, the latest stable version (0.8.1) has
a bug which prevents orisync from working properly.

The problem is that orisync did not do its work, aka. synchronizing the two
repositories.

We followed the steps exactly as described in the official paper and re-created
each repository numerous times, but it did not change anything.

So, when for example, creating a new directory, that directory is not created on
the other side even after having waited for 1+ minute.

This bug cost us about 3 hours of working time each.

Fortunately, the latest developer version, called ori-orisyncng, fixes that
problem.

It is available in the following git repository:

.. code:: text

    https://bitbucket.org/orifs/ori-orisyncng.git                                   


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

======================== ===================== ===================== =============
 -                       **Ori File System**   **HDFS**              **GlusterFS**
======================== ===================== ===================== =============
**Use case**             Replication; in       Big data;
                         future bidirectional  write-one-read-many
                         synchronization
**Architecture**         Peer to Peer          master/slave [7]_
**Replication Strategy** polling (every 5      Replicates created at
                         seconds)              the time of writing
                                               [6]_
**Security**             Transmission via SSH; None [6]_
**Consistency**          merkle tree
======================== ===================== ===================== =============

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

.. _8:

[8] orifs / ori / wiki / Home &mdash; Bitbucket,
    https://bitbucket.org/orifs/ori/wiki/Home,
    visited: 2015-03-19

.. header::

    +-------------+--------------------+------------+
    | ###Title### | Andreas Willinger, | 2015-03-19 |
    |             | Jakob Klepp        |            |
    +-------------+--------------------+------------+

.. footer::

    ###Page### / ###Total###
