# Network Time Protocol (NTP) and Network File System (NFS)

*Originally created by Annie Christy*

Note: Most of the commands in this lab need to be run as the superuser,
so consider running them as root.

## Network Time Protocol (NTP)

### What is NTP?


NTP is an internet protocol that uses packet-switched, variable-latency
data networks to synchronize system clocks. The purpose of NTP is to
synchronize the clocks of all connected computers within a few
milliseconds of Coordinated Universal Time, which is also known as UTC.

NTP clocks are organized into strata, where stratum 0 is made up of
extremely precise atomic clocks, GPS clocks, or radio clocks. Stratum 1
is made up of computers that are coordinated with the atomic devices.
The lower strata (2 through 15) form a tree of synchronized servers that
communicate with one another to achieve synchronization.

We are interested in this protocol because some of the services we want
to run depend on servers and clients that have exactly synchronized
time. Specifically, we're going to set up a networked file system (NFS) that
requires the client and server clocks to be synchronized.

For more information, visit
<http://en.wikipedia.org/wiki/Network_Time_Protocol>

### Enabling NTP


The class server (`crispy.sys.cs.hmc.edu`) is already running NTP. Your goal is 
to use NTP to sync *your* system's clock to the server’s clock.

You want to tell your computer to query NFS server for NTP
synchronization. To do this, open the /etc/ntp.conf file and add a line
directing the NTP implementation to the class server.

    sudo vim /etc/ntp.conf

Add the following lines to the section whose comments tell you to
"Name of the servers ntpd should sync with."

    server          crispy.sys.cs.hmc.edu
    server          127.127.1.0
    fudge           127.127.1.0          stratum 10

The first line instructs ntpd to sync with the class NFS server. The
following two lines tell ntpd to synchronize with itself if it loses its
connection with the class server.

_Aside_: Note the presence of a `driftfile` setting in `/etc/ntp.conf`. This file is where
the machine can store information about the natural time drifts that
your computer has, so that it can automatically adjust for them.

You need to uncomment the bottom line in the file to ensure that other
machines on your network can synchronize with your server, but that they
cannot configure the server.

    restrict <subnet> mask 255.255.255.0 nomodify nopeer notrap

(*Note:* The `<subnet>` field is the network for our machines; it will be provided in class.)

The NTP service has a function called monlist, that is used to query the
server about the hosts that have connected to it. This can be useful if
you want to learn about your network stats, but we don’t really need it
and it leaves your machine susceptible to denial-of-service attacks. To
disable this functionality, add the following line to the bottom of the
/etc/ntp.conf file:

    disable monitor

You can now save and close the file.

NTP runs as a service. You can see which services are running on your system
with the command:

    rc-status

To start the NTP service and check that it is running, run the following
rc-service commands as root.

    sudo rc-service ntp-client start

Verify that the ntp-client service has started.

Add the service to your "default run level" so that NTP will start at
boot.

    sudo rc-update add ntp-client default

To verify that the ntp client runs every time the machine restarts, restart your
machine!

    sudo reboot

Once the machine is back up, check that the ntp client is running.

For more information, visit <http://wiki.gentoo.org/wiki/Ntp>

## Network File System (NFS)

### What is NFS?


In 1984, Sun Microsystems developed a distributed file system protocol
called a Network File System (NFS). NFS makes it possible for clients to
access files on a remote server as though they were a part of the local
file tree. The class server is already running the NFS service. It is
running the NFS daemon, nfsd, and it will reference the file
/etc/exports to determine what filesystems it is allowed to export to
clients. We have already added your computers to this file and given them
read-write access to the /local directory. Your first goal will be to
mount the server's /local directory on your machine.

To access the NFS shares on the server, we will use tools from the
nfs-utils package, which has already been installed.

<!--
Start the rpc.statd daemon.

    sudo rc-service rpc.statd start
-->

Start the nfs service.

    sudo rc-service nfs start

For more information, visit
<http://en.wikipedia.org/wiki/Network_File_System>

### Manually Mounting NFS Shares


We will begin by learning how to manually mount the desired filesystems
onto your machine.

You need to create the directory that you want the NFS share to mount
to. It doesn’t really matter what you call this directory, but it makes
sense to call it by the same name that it has on the server. So, create
a file called /local.

    cd /
    sudo mkdir /local

Just to convince you that the filesystem is not mounted yet, please go
into /local. It should be empty.

    cd /local
    cd -

Now, mount the filesystem from the server. This command takes the form of
mount server.address:/server/path /client/path.

    sudo mount crispy.sys.cs.hmc.edu:/export/sysadmin /local

Now, if you go into the /local file, you should see several folders.
Open the file called README and follow the instructions there.

It is good to know how to manually mount filesystems, but it is not a
particularly efficient way to work if you want to use the NFS shares a
lot because the share becomes unmounted when the computer is rebooted.
To prove this to yourself, you should reboot your computer now (but
first make sure that you don’t have any important processes running).

    sudo reboot

You will lose your connection. Wait for a bit, then
try to ssh back in.

Now, go back to the /local directory. It should be empty because your
machine isn’t getting the files off the NFS server anymore. You don’t
need this directory anymore, so you can delete it.

    rmdir /local

### Mount NFS Shares at Boot with autofs


You may have noticed that if you change something in your home directory
on knuth, that change will appear when you log into your home directory
on any of the other machines on the CS Department cluster. This works
because the home directories are exported to the machines using NFS.
However, it is unlikely that you have ever had to mount your home
directory on knuth before working with it. That would be really annoying
because you would need to do it every time you wanted to access your
files! Instead, it is possible to mount the NFS shares automatically
when the computer boots. That way, the NFS files that you use a lot
will be waiting for you when you log on.

#### Configure autofs

We use a program called autofs to automount files. There are two files
that we need to configure on the client machine. First, read through the
file /etc/autofs/auto.master.

    sudo vim /etc/autofs/auto.master

This file tells autofs where to mount files and also provides a path to
a file that provides details about what to mount. At the bottom of the file,
uncomment (i.e., remove the `#` from the begging of) the line that says 

  ```
  +auto.master
  ```

Add the following line to the file beneath the comment that says ’For details of
the format look at autofs(5)’ but make sure the line you add is uncommented.

    /mnt    /etc/autofs/auto_mnt  --ghost

The line you just added to the file tells the computer to mount the NFS shares
in the /mnt directory. The `--ghost` option causes the system to create empty
folders for the mountpoints. The line you added also gives the the path to the
automnt file which will contain a list of the NFS shares that should be mounted
automatically. Open that file now.

    sudo vim /etc/autofs/auto_mnt

This file should be empty when you first open it. Any line you add will
represent a directory that you want to mount to your system. The first
part of a line is the name of the share as you want it to show up in
your /mnt directory. the second part of a line provides the location of
the share on the NFS server. Tell the machine to automount the folder from our
server by adding the following line.

    sysadmin    -rw,soft,intr,async    134.173.40.190:/export/sysadmin

#### Test autofs

Now start the autofs service, which will auto-mount directories on-demand

    rc-service autofs start

Now, navigate to the following directory

    cd /mnt/sysadmin

If everything is configured correctly, you should now be in the shared, NFS-
mounted directory!

#### Start autofs on boot

If you want autofs to mount the NFS share at boot, then you need to make sure
that autofs itself is running at boot. Make sure that nfs and autofs are all
added to the default run level.

    sudo rc-update add nfs default

    sudo rc-update add autofs default

The ntpd service should already be on the default run level, but you can
use the rc-status command to check. It will print a list of all the
services that start at boot.

    rc-status

Now, reboot your machine. You will lose your connection. Wait for a bit, then
try to ssh back in.

    sudo reboot

The files should be mounted automatically **on-demand** (i.e., when you first
navigate to the file / directory). Check to make sure.

    cd /mnt/sysadmin

## Choose your own adventure
Here are some projects to work on. They're pretty ambitious, and you probably
won't finish them before the end of class. Get as far as you can on one or more
of them, then write about your experience when you fill out today's survey.

   1. See if you can get NFS to mount the remote file system automatically, when
   you start the computer. You'll want to use [autofs](http://wiki.gentoo.org/wiki/AutoFS)

   1. Find a partner and see if you can use each other's machines as NFS 
   *servers*. Your partner should be able to mount files hosted on your machine,
   and vice-versa. 

   1. If you're able to turn your machine into an NFS server, can you get a 
   *non*-sysadmin machine (e.g., your laptop) to mount the remote file system?

   1. See if you can mount your knuth home directory on your sysadmin machine. 
   (I have no idea if this is possible; I suspect it isn't, but can we find out
   why?)

   1. NTP has some [historically notable weaknesses](https://en.wikipedia.org/wiki/NTP_server_misuse_and_abuse).
   Is our NTP server vulnerable to abuse? If not, why not? If so, can you prove
   it :)? (Warning: be very very careful about this. Do not create a lot of
   network traffic, i.e., do *not* initiate a denial-of-service attack.)

