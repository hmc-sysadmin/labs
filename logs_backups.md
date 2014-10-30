# Logging and Backups
*Originally created by Paige Rinnert*

In this lab, we'll explore ways of storing and investigating the past behavior
of our systems.

## Logging 

Many logging servers use syslog, and since almost all machines use it,
it is easy to to sync the logging of multiple devices. Once a syslog
server is set up, devices can send their log messages over the network
to the server rather than recording them in a local file or displaying
them. Doing this creates a central logging server that becomes a
repository for log messages.

A severity level can be used to decide which messages are sent. The
levels are standardized and identified by a number and/or a standard
abbreviation:

 | --------- | ---------------- | --------- |
 | Number    | Description      | Abbr.     |
 | --------- | ---------------- | --------- |
 | 0         | Emergency        | emerg     |
 | 1         | Alerts           | alert     |
 | 2         | Critical         | crit      |
 | 3         | Errors           | err       |
 | 4         | Warnings         | warn      |
 | 5         | Notification     | notice    |
 | 6         | Information      | info      |
 | 7         | Debug            | debug     |
 | --------- | ---------------- | --------  |

There are also things called "facilities" which loosely relate to
system processes, a way of categorizing messages. When a remote device
sends a message to a syslog server it includes one of the standard
facility values (along with a severity level). Some of the common
facilities are:

 | ------------------- | ----------------------------------------------- |
 | Code                | Explanation                                     |
 | ------------------- | ----------------------------------------------- |
 | auth                | authentication (login) messages                 |
 | cron                | messages from the memory-resident scheduler     |
 | daemon              | messages from resident daemons                  |
 | kern                | kernel messages                                 |
 | lpr                 | printer messages (used by JetDirect cards)      |
 | mail                | messages from Sendmail                          |
 | user                | messages from user-initiated processes/apps     |
 | local0 - local7     | user-defined                                    |
 | syslog              | messages from the syslog process itself         |
 | ------------------- | ----------------------------------------------- |

One can specify different severity levels for different facilities so
one can, for example, log all kernel messages but only emergency
messages from printers. The logs can then be viewed on the central
server. How a server is set up differs with the main operating system
being used.

### Using sysklogd 

Your system will be running `sysklogd`. The file used to configure syslog
is `/etc/syslog.conf`. In addition to sysklogd, your system is running
`logrotate`. This program determines how long a system saves old logs and how
old and new logs are managed.

If you’re interested in the different kinds of loggers that you can use
on a Gentoo system, you can read more about them on the Gentoo website:
<https://www.gentoo.org/doc/en/security/security-handbook.xml?part=1&chap=3>.

### Logs found in /var/log 

Run `ls` in the directory `/var/log`. These are the 
[logs maintained by the system](http://www.thegeekstuff.com/2011/08/linux-var-log-files). 
Usually you need
to use sudo to view log files. (Since logs tend to be
thousands of lines long, here is a helpful hint for viewing them: in
vim, typing SHIFT-G will take you to the bottom of a file while typing
gg will return you to the top. Also don’t forget that you can search
files.)

1.  `auth.log` contains authorization records, including logins and uses of `
su

    Look at your auth.log file. You should see some lines like this,
    among others. The line below is a record of using the sudo command
    to view this file, auth.log, from the directory /var/log
    (`PWD=/var/log`) using the command vim (`COMMAND=/usr/bin/vim`).

        Jul  2 10:09:56 penguin sudo: prinnert : TTY=tty1 ; PWD=/var/log ; 
        USER=root ; COMMAND=/usr/bin/vim auth.log

    **Open your auth.log file and see if you can find some of the commands
    you have recently performed using sudo.**

2.  `emerge.log` contains a record of every package emerged on the
    system, whether the emerge finished successfully or not.

    Here is a sample output of a package emerge (this example is
    emerging telnet):

        1413356210:  *** emerge  cowsay
        1413356231:  >>> emerge (1 of 1) games-misc/cowsay-3.03-r2 to /
        1413356232:  === (1 of 1) Cleaning (games-misc/cowsay-3.03-r2::/usr/portage/games-misc/cowsay/cowsay-3.03-r2.ebuild)
        1413356232:  === (1 of 1) Compiling/Merging (games-misc/cowsay-3.03-r2::/usr/portage/games-misc/cowsay/cowsay-3.03-r2.ebuild)
        1413356236:  === (1 of 1) Merging (games-misc/cowsay-3.03-r2::/usr/portage/games-misc/cowsay/cowsay-3.03-r2.ebuild)
        1413356238:  >>> AUTOCLEAN: games-misc/cowsay:0
        1413356239:  === (1 of 1) Updating world file (games-misc/cowsay-3.03-r2)
        1413356239:  === (1 of 1) Post-Build Cleaning (games-misc/cowsay-3.03-r2::/usr/portage/games-misc/cowsay/cowsay-3.03-r2.ebuild)
        1413356239:  ::: completed emerge (1 of 1) games-misc/cowsay-3.03-r2 to /
        1413356239:  *** Finished. Cleaning up...
        1413356240:  *** exiting successfully.
        1413356241:  *** terminating.

    However, below is an example of an entry in the emerge log after
    attempting to emerge a package that doesn’t exist. This is an
    example of trying to emerge telnet without using the correct package
    name.

        1403904498: Started emerge on: Jun 27, 2014 14:28:18
        1403904498:  *** emerge  telnet
        1403904499:  *** exiting unsuccessfully with status '1'.
        1403904499:  *** terminating.

    Note that the program exited unsuccessfully with status 1. A
    successful command will always exit with status 0 and any other exit
    status indicates an error. Usually the status 1 is used as a
    catch-all for all general errors.

    **Scroll through your emerge.log file and look for some of the
    packages you’ve installed recently.**

3.  `mail.log` contains a record of the emails you have sent and
    received.

    Sample output in mail.log when an email was sent from the computer
    penguin to the computer heisenberg:

        Jul  2 10:38:48 penguin postfix/pickup[20871]: C8762CA377D: uid=1000 
        from=<root>
        Jul  2 10:38:48 penguin postfix/cleanup[21281]: C8762CA377D: message-id=
        <20140702173848.GA21269@penguin.sys.cs.hmc.edu>
        Jul  2 10:38:48 penguin postfix/qmgr[12656]: C8762CA377D: from=<root
        @sys.cs.hmc.edu>, size=533, nrcpt=1 (queue active)
        Jul  2 10:38:49 penguin postfix/smtp[21283]: C8762CA377D: to=<root@
        heisenberg.sys.cs.hmc.edu>, relay=heisenberg.sys.cs.hmc.edu[134.173.43.69]
        :25, delay=0.64, delays=0.17/0.06/0.32/0.08, dsn=2.0.0, status=sent 
        (250 2.0.0 Ok: queued as 74E63AE0434)
        Jul  2 10:38:49 penguin postfix/qmgr[12656]: C8762CA377D: removed

    **Send an email to a friend’s computer using either sendmail or mutt,
    and have them send you an email. Go to your mail log and look at
    what appears in the log when you send an email and when you receive
    one.**

4.  `user.log` contains a record of when your computer has been shut
    down or rebooted.

### Compressed Files 


Notice that there are a bunch of zipped files in /var/log. These files all end 
in the extension `.gz`, showing that they are
zipped (compressed) files.

The system uses compressed files because they take up less disk space.
The programs that zip and unzip files are called `gzip` and `gunzip`,
respectively. Try unzipping one of the zipped log files by running the
following command. You will probably need to sudo.

    gunzip <filename>

Now run `ls` in /var/log. You will see that the file you just unzipped
has lost the extension .gz. There is
really no reason to have it unzipped though, so go ahead and re-zip it
with this command:

    gzip <filename>

Now that our zipped log file is back to how it was when we found it, run
the following:

    ls -l /var/log/auth*

You should see the current auth log file (`auth.log`) as well as any
zipped versions of the file. Compare their filesizes. The zipped files
may be up to one-tenth the size of the regular, uncompressed log files.
Usually compression techniques work by eliminating redundant data or
expressing it in a more succinct way.

Each of the red, zipped files in /var/log is an old log that is
compressed to save space. The logger syslogd automatically compresses
them because it makes sense to compress older log files that you most
likely won’t need. However, it is good to keep these files rather than
just deleting them because they may be useful if you have any security
breaches or mysterious errors.

### Remote Logging 

You may have noticed while perusing your log files that it is very easy
to retroactively make changes to a log file or even delete information.
It would be entirely possible to delete lines in a mail log so that it
looks like a message was never sent, or go into the auth log file and
erase a series of commands performed with sudo. One way to protect your
computer against log file alteration by a hacker in covering their
tracks is by enabling remote logging.

Remote logging allows a computer to log information on another server as
well as in its own log files. This means that if information is deleted
from the log files on the computer, there is still a record of them on
another computer. These files could be compared to determine if any
alterations were made to the log files and what the changes were.


**Pick a partner (or form a larger team) for remote logging.**

Complete the following steps to enable somone else to store their logs on your
machine:

1. The syslogd program needs the -r flag enabled, to tell it to listen for log
   events on the network. You can set this flag on the command
   line when you start sysklogd (like this: `syslogd -r`) but this is
   not permanent and requires you to kill syslogd and then start it
   again from the command line. A better option is to edit the file
   `/etc/conf.d/sysklogd` and add the -r flag to the `SYSLOGD`
   variable. The file should now look like:

        SYSLOGD="-m 0 -r"
        KLOGD="-c 3 -2"

1.  Since the default behavior of syslogd is that it does not listen on
    the network, we need to make sure that syslog is set up to listen on
    the standard port in `/etc/services`. Search for the number 514 and
    make sure the following line is in the file. If it’s not there, add
    it.

        syslog  514/udp

1. Restart system logging by running the following command:
        
        sudo /etc/init.d/sysklogd restart

Here are the steps to export your logs to another system:

1. Get the hostname of your friend's machine, i.e., the one to which you'll send 
   your logs. The hostname should have the form `vm#.sys.cs.hmc.edu`.

1.  Open the file `/etc/syslog.conf`. This file contains lists of the
    various logs kept by the system. Add your friend's hostname of a friend’s
    computer near the top of the file with this syntax:

        *.* @hostname.domainname

    The \*.\* causes all messages to be logged on the remote host. (If
    you only wanted one type of message to be logged on the remote host,
    for example kernel messages, you would replace \*.\* with kern.\*)

1. Restart system logging by running the following command:
        
        sudo /etc/init.d/sysklogd restart

Now you and your partner’s computers should contain the same log files!
If you are both using remote logging to send messages to each other’s
computers, the log files on both will contain the same information. If
information is deleted out of one of the log files, it will still be
present in the other log file. You could then use the diff command to
compare log files if a discrepancy arose. This makes your computer safer
because if a hacker was trying to cover their tracks, they would have to
edit not only the log file on your machine, but also determine that the
logs were being saved remotely and delete logs off of the other machine.
Possible, but more unlikely.

## Important! 

Before moving on, run the following command:

    sudo /root/MysteriousScript1

## Backups 

Backups are an important part of any system, as you may well know if you
have ever had your computer crash on you. Saving your files in backups
often will save you if anything happens to your system. This is
especially important when we are doing so much messing around with our
system, because it allows us to restore our files if we screw something
up.

The CS department uses the program AMANDA for our backups (AMANDA stands
for the **A**dvanced **M**aryland **A**utomatic **N**etwork **D**isk
**A**rchiver). It was originally developed at the University of
Maryland’s computer science department and is now a widely used backup
system that is completely open source.

### Making a backup 


1.  The first step now is to create some directories. Create the
    directory /amanda, and then change its owner to amanda.

        sudo mkdir -p /amanda
        sudo chown amanda /amanda

    Now you will need to switch to the amanda user. Before you can do
    this, though, you need to change the password for the amanda user.
    **We will provide a password for everyone to use, in lab. Ask for the
    password when you get this point.**

        sudo passwd amanda

    Now you can `su amanda` and create more directories. (Note that the
    -p flag will create the parent directories if they don’t already
    exist. Also, the curly braces will create multiple directories in
    the same parent directory).

        mkdir -p /amanda/vtapes/slot{1,2,3,4}
        mkdir -p /amanda/holding
        mkdir -p /amanda/state/{curinfo,log,index}
        mkdir -p /etc/amanda/MyConfig 

2.  Run the following commands to retrieve a configuration file for your backups

        cd /etc/amanda/MyConfig/        
        wget http://www.cs.hmc.edu/courses/current/sysadmin/amanda.conf

3.  Open this config file and change the mailto configuration. Also,
    browse through the file. Notice that the files created in
    the previous step are mentioned throughout the config file. Each
    backup will have its own folder and config file. If you want to, you
    can look at the example amanda.conf file that comes with the amanda
    install in /etc/amanda/DailySet1.

3.  In the directory MyConfig, where you just created the config file
    amanda.conf, create the file `disklist` and put in the line below.
    All lines in the disklist have the format
    `<hostname> <diskname> <dumptype>`. The hostname here is localhost
    and the disk to back up is /home. The dumptype used here is the one
    that we just created in the config file.

        localhost /home simple-gnutar-local

4.  Now we need to check that the configuration worked correctly. Since
    the amanda commands are all located in /usr/sbin, we want to add
    this to the `PATH` variable for amanda so that we can run the
    commands without including the path every time. As the user amanda,
    run `echo $PATH`. If /usr/sbin is not there, add this line to the
    `.bashrc` file in amanda’s home directory (you may need to create
    the file if it doesn’t exist):

        export PATH=$PATH:/usr/sbin

    To reference this file, now run the following command. Subsequent
    times, your updated path will be automatically added to the `PATH`
    variable on restart.

        source ~/.bashrc

    Now check the path variable again, and you should see /usr/sbin. Now
    you can run:

        amcheck MyConfig

    The output should look something like this:

        Amanda Tape Server Host Check
        -----------------------------
        NOTE: tapelist will be created on the next run.
        Holding disk /amanda/holding: 868352 kB disk space available, using 51200 
        kB as requested
        slot 1: contains an empty volume
        Will write label 'MyData01' to new volume in slot 1.
        NOTE: skipping tape-writable test
        NOTE: host info dir /amanda/state/curinfo/localhost does not exist
        NOTE: it will be created on the next run.
        NOTE: index dir /amanda/state/index/localhost does not exist
        NOTE: it will be created on the next run.
        Server check took 0.276 seconds

        Amanda Backup Client Hosts Check
        --------------------------------
        Client check: 1 host checked in 7.089 seconds.  0 problems found.

        (brought to you by Amanda 3.3.3)

    If there are any extra lines in your output that look like they
    might be errors, you might need to resolve these before moving on.

5.  Now you can run your first backup!

        amdump MyConfig

    The command should take a few seconds to run, that’s probably a good
    thing. When it’s finished, try `echo $?`. If the result is anything
    other than 0, it means that amdump ended in an error state and did
    not run correctly. If it is 0, you’re golden.

6.  Now run this command:

        amreport MyConfig > /amanda/amandareport

    This command puts the report from running the backup into a file in
    the amanda directory. Go view the file you just created. This is the
    output that gets emailed to CS staff whenever amanda runs a backup
    on Diffie.

Congratulations, you’ve finished your first backup! Now let’s figure out
how to restore the files that have been backed up.

### Restoring a backup 


1.  There are a few changes you need to make to the amanda client
    configuration file,
        `/etc/amanda/amanda-client.conf`. Change the lines in this file to
    match the ones below:

        conf "MyConfig"
        index_server "localhost"
        tape_server "localhost"
        tapedev "changer"
        auth "local"

    Don’t worry about the `ssh_keys` line, just leave it as the empty
    string.

2.  Now we need to create a place where the backup will restore to. Go
    to `/tmp` and create the directory `test-recovery`. Go to that
    directory.

        mkdir /tmp/test-recovery
        cd /tmp/test-recovery

3.  Now you’re ready to restore the files. Since you need permission to
    access files on the system, `amrecover` needs to be run with sudo.

        amrecover MyConfig

    You should see something similar to the following. Now amrecover
    will prompt you to enter information.

        Using index server from environment AMANDA_SERVER (penguin)
        AMRECOVER Version 3.3.3. Contacting server on penguin ...
        220 penguin AMANDA index server (3.3.3) ready.
        Setting restore date to today (2014-07-08)
        200 Working date set to 2014-07-08.
        200 Config set to MyConfig.
        501 Host penguin is not in your disklist.
        Trying host localhost ...
        200 Dump host set to localhost.
        Use the setdisk command to choose dump disk to recover
        amrecover>

    Lines with errors begin with a 5 while the lines that work correctly
    begin with a 2. If the program complains that the host is not in
    your disklist, set the host to localhost by typing the following.
    Otherwise, keep going.

        amrecover> sethost localhost

4.  Now set the disk to /home, the directory you backed up.

        amrecover> setdisk /home

    Amrecover should confirm that it did the proposed action by printing
    the following:

        200 Disk set to /home.

5.  And add the name of your home directory (should be your user name):

        amrecover> add <directory name>

    Again, amrecover should confirm that it added the directory:

        Added directory <directory name>

6.  Now you’re ready to extract files.

        amrecover> extract

    Amrecover will ask you to confirm what you want to do, and if it
    looks right, type Y (make sure it is a capital Y or it will get
    angry with you).

        Extracting files using tape drive changer on host localhost.
        The following tapes are needed: MyData01

        Extracting files using tape drive changer on host localhost.
        Load tape MyData01 now
        Continue [?/Y/n/s/d]? Y

    Amrecover should list the recovered files, returning you to the
    amrecover prompt. If so, you have successfully restored your home
    directory! That is the end of the recovery process, so type CTRL-D
    to exit amrecover and return to your command prompt.

    If you run `ls` in your current directory, `/tmp/test-recovery`, you
    should see a copy of your home directory.

### Backups: Activity


Now that you’ve successfully backed up your home directory, let’s back
up more files using the same process. Add a new configuration called
MyLogs to back up the disk `/var/log`. To start out, as the amanda user
copy the MyConfig directory and name the new directory MyLogs. You will
need to change the name of the configuration in
`/etc/amanda/MyLogs/amanda.conf` and `/etc/amanda/amanda-client.conf`,
and change the disk to back up in `/etc/amanda/MyLogs/disklist`. Then
run `amcheck MyLogs` to check the new configuration and `amdump MyLogs`
to actually perform the backup.

### Resources 

<http://wiki.zmanda.com/index.php/GSWA/Build_a_Basic_Configuration>

<http://wiki.zmanda.com/index.php/GSWA/Recovering_Files>

## Important! 

Before continuing, run the following command:

    sudo /root/MysteriousScript2

## Cron Jobs 


Backups are so important that you should do them often, at regular
intervals. It can be difficult to remember to do backups, so that is one
of the reasons that cron jobs exist. Cron jobs automate processes so
they will run at set intervals, which is perfect for keeping a system’s
backups up-to-date.

The cron jobs running on your system are listed in the `/etc/crontab`.
Go take a look at the file. Here is a sample line from this file:

    19 4 * * 0,3,6  root    rm -f /var/spool/cron/lastrun/cron.weekly

The last part of the line is fairly straightforward. `root` is the user
running the command, and following root is the command itself. The
beginning of the line may look ambiguous, but it describes how often the
job should run. The numbers represent:

    19        4        *                *        0,3,6
    minutes   hours    day (of month)   month    day (of week)
    (0-59)    (0-23)   (1-31)           (1-12)   (0-6)         # allowable values

So, this particular command removes the file `cron.weekly` at 4:19am
every Sunday, Wednesday, and Saturday.

### Cron Jobs: Activity

Add two lines to the crontab file to check the configuration of MyConfig
at 9pm and run a backup every day at 9:30pm. Set up amcheck so that
instead of printing to standard output like it usually does, it mails
the output to root. (Reminder: the amanda commands have man pages just
like any other command!)

## SysAdmin: Activity
Check the root's mail.
