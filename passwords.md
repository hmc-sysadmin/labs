[BadPasswords]: https://splashdata.com/press/worst-passwords-of-2014.htm

# Passwords and Encryption
**Originally created by Lisa Goeller**

## Introduction 

A weak password can create a hole in an otherwise completely secure
system. All it takes is one user with a simple password and someone who
knows their way around UNIX to compromise your system. As people
continue to create more sophisticated password crackers, it's becoming
easier and easier for passwords to be guessed. It is incredibly
important to make sure that your users have secure passwords to limit
security threats as much as possible.

It may seem hard to believe, but people will still choose weak passwords
if given the option. SplashData, a company that manages passwords, posts
its most common passwords every year. Take a look at the most common
passwords of 2014, according to [SplashData][BadPasswords]

   1. 123456    
   1. password  
   1. 12345     
   1. 12345678  
   1. qwerty    
   1. 123456789 
   1. 1234      
   1. baseball  
   1. dragon    
   1. football  

As you can see, weak passwords are still VERY prevalent on the
interwebs. In this lab, we will explore what you can do with passwords
and encryption.

## Passwords (and how to crack them)


The passwords on your system are stored in the `/etc/passwd` and
`/etc/shadow` files. The `/etc/passwd` file is world readable because
many commands, like `ls`, rely on it to display file ownerships and
similar characteristics. The `/etc/passwd` contains account information
and stores passwords as a single ‘x' character while the `/etc/shadow`
file, which only gives root read and write permissions, contains the
encrypted passwords and personal information about the user. Both files
store this information in a single line of code for each user in their
respective files.

User accounts are not only limited to humans. If you take a look at
either your `/etc/passwd` or `etc/shadow` files, you'll see that they
contain more user accounts than there are human users on your machine.
Many commands, like `amanda` need to have a user account in order to
execute properly.

If you'd like to learn more about the formats of these files, 
[this is a good resource](http://tuxthink.blogspot.com/2012/03/look-into-etcpasswdetcshadow-and.html)

### John the Ripper 


John the Ripper is a password cracking tool used by some administrators
to determine the strength of their users' passwords. John checks your current 
passwords against its preloaded list of common passwords to
see how secure your system's passwords are.

John can run in regular, single, wordlist,
and incremental modes. Single mode uses a dictionary
created out of a user's *gecos fields*, entries in the `/etc/passwd` file
that contain general information about users like their real name and
contact information. In incremental mode, John will try all possible
variations of ASCII characters for passwords up to 13 characters long.
Just as a warning, John can take up a lot of CPU processing time and it
can sometimes take a couple days to crack a password.

Note: Many have been arrested for using John the Ripper on a network.
Make sure before running password crackers to always let the higher ups
know if you are not doing it on your own machine.

Now that you know a bit about John, try it out yourself!

Navigate to the nfs-mounted directory `/mnt/sysadmin/passwordsLab`. In it, you 
should see a file called jackson.txt in your home directory. Take a
look at that file.

Recall from the readings what each of these fields are. What can you
tell about mike's account just from looking at this files? Run John
against the jackson.txt file.

    /usr/sbin/john jackson.txt

John will soon let you know it's completed its job. By default, John
runs through single crack mode first, then uses a wordlist with rules,
and finally goes for incremental mode. The first time John cracks a password, it
will display the result. It also stores the result in the `.john/john.pot` file
in your home directory. To show them again, you can run:

    /usr/sbin/john --show jackson.txt

Now you know mike's password---it's one of my favorite tunes!

**Note:** For obvious reasons, there is not an actual user named mike with this
password on your system.

### Choosing a Strong Password 

You're probably familiar with the basic requirements of a strong
password. It's generally accepted that a password should have a minimum
length of eight characters and contain at least one number, one
non-alphanumeric character, and one letter. It can be difficult coming
up with passwords on your own. The 
[xkcd password generator](https://xkpasswd.net/s/) is a nice tool for doing so. 

Gentoo also offers some tools for random
password generation. We're going to look at one of them, `passook`.

You can use `passook` to generate some random passwords. The general
syntax for `passook` is as follows:

    passook [-n num] [-p plev] [-u usernames]

where `num` is the number of passwords you want to generate, `plev` is
the pronunciation level, on a scale from 1 (few alphanumeric
characters) to 5 (all alphanumeric characters), and `usernames` is
the path to the usernames file. If the `usernames` option is included,
`passook` will automaticlaly add the passwords it generates to the
usernames in the file.

For funsies, try generating a couple passwords with different
pronunciation levels.

### pam and cracklib

Now that you've seen what strong passwords look like, it's time to
ensure that all the passwords on your system will meet your own password
requirement, whatever that may be.

`pam`, the `pluggable authentication module`, is used for authentication
between different services. `pam` acts as a layer between authentication
services and the programs that need authentication before they can
proceed. It's used by tons of applications. Run the command below to see
the list of applications with `pam` support.

    sudo ldd /{,usr/}{bin,sbin}/* | grep -B 5 libpam | grep '^/'

You can also see the list of packages that depend on `pam`:

    equery depends pam

As you can see, `pam` is a very important part of your system. `pam`'s
functions can be roughly divided into four jobs:
authentication, account, session, and password. Authentication makes
sure that the user has valid credentials, account checks whether the
user has permission to access whatever they're trying to do, session
builds up the environment, and password is for updating and modifying a
user's account information.

`cracklib` is an add-on feature that can be used to enhance
`pam's authentication` function, forcing users to create stronger
passwords. We will explore some of these actions now.

1.  Go to `/etc/pam.d/passwd` and edit it so that it somewhat resembles
    the code below.

        auth     required pam_unix.so shadow nullok
        account  required pam_unix.so
        password required pam_cracklib.so retry=3 minlen=8 enforce_for_root
        password required pam_unix.so md5 use_authtok
        session  required pam_unix.so

    These are just some example options. The `retry=3` command gives the
    user three tries to get a proper password and `minlen=8` requires a
    password that is at least eight characters long. Normally these
    options only enforce these requirements for normal users. Adding the
    `enforce_for_root` option forces the root user to follow the
    password requirements as everyone else. To see more options, simply
    type `man pam_cracklib` into the command line.

2.  Create a fake user `maninthemirror`.

        sudo useradd maninthemirror

    Now try giving `maninthemirror` a password.

        sudo passwd maninthemirror

    You'll notice that if you try to enter a password that doesn't meet
    the minimum requirements that you described in your
    `/etc/pam.d/passwd` file then you will be unable to set
    `maninthemirror's` password. `cracklib` is thus a very powerful tool
    to secure your system.

    (Remember, the `userdel` command removes users, if you don't want to keep
    your new user around.)

### Aside: password dictionaries 


`cracklib` comes equipped with a simple dictionary, `cracklib-small`.
This is a basic set of words to run your potential passwords against. On
the `cracklib` download page, there are plenty of other wordlists that
you can download and use to test the strength of your passwords. All of
your dictionaries are automatically stored in `/usr/share/dict`,
including the `words` dictionary that comes as a default on all Unix
systems and is used by word processors like `vim` for spellchecking.
If you add another dictionary, you can have cracklib include this
dictionary and all the other dictionaries when checking password
strength simply by typing this command:

    sudo create-cracklib-dict /usr/share/dict/*

`create-cracklib-dict` takes wordlist files and converts them into
`cracklib` dictionaries so that `cracklib` can use them. `cracklib`
creates three files in your `/usr/lib` directory:
`cracklib_dict.hwm, cracklib_dict.pwd` and `cracklib_dict.pwi`. These
are the files that cracklib looks up whenever it's doing a check.

When you run `create-cracklib-dict` you'll see that two numbers are
displayed. The first is the number of words that `cracklib` found in
your wordlist. The second is the number of words from that file that it
added to its dictionary.

## Encrypting Files 

We've seen how to use UNIX permissions to restrict access to files. 
There is another way to protect a file: encryption. There
are a couple times when you should choose to encrypt files instead of relying on
UNIX permissions. Any personal information on your computer like financial
records or your social security number should definitely be encrypted.
You should also encrypt files if you store files that have business
information and company secrets on your computer. In this lab, we'll
look at encrypting files using `openssl` or `gnupg`.

### Types of Encryptions 


It may be helpful for you to see what types of encryption algorithms you
already have on your system. These are the ciphers included on your
kernel.

    cat /proc/crypto | grep name

The US standard for data encryption is `aes`. `des`, the
`Data Encryption Standard`, was the original standard of the National
Institute of Standards and Technology. As time passed though, people
began to see the flaws of `des`: namely that `des` used only a 56-bit
data encryption key. In 1999, Deep Crack and distributed.net broke a
`des` key in 22 hours, revealing how weak `des` encryptions were. Since
then, `des` has been replaced by `aes`, the
`Advanced Encryption System`. This new algorithm uses a 128-bit block
size, but has key sizes that can be either 128, 192, or 256 bits. There
are many other encryption algorithms out there.

In the next section, we will be using OpenSSL. Take a look at what
encryption algorithms are prebuilt into OpenSSL.

    openssl ciphers -v

### OpenSSL 


OpenSSL implements the TLS and SSL protocols and doubles as an
encryption library. OpenSSL allows users to securely encrypt files. To
ensure that everything functions properly and no errors arise, it's best
to have an ingoing and outgoing destination for your encrypted file. The
OpenSSL encryption syntax looks like this:

    openssl [encryption type] -in file-to-encrypt -out encrypted-file

Try it out for yourself. In your home directory, create a file `file.txt` and 
write a secret message inside. Encrypt the file using OpenSSL.

    openssl enc -aes256 -in file.txt -out /mnt/sysadmin/passwordsLab/secrets/<name>_encrypted.txt

where `<name>` is your user name.

This will encrypt your file using AES with a 256-bit key. You'll be prompted
to enter a passphrase. This is the phrase you'll use to decrypt the
file, so don't forget it! Once you've done this, `rm file.txt`, the old
file, and `less encrypted.txt`. You'll see you can't read your secret
now that your file has been encrypted. Looks like you'll have to decrypt
the file.

    openssl enc -d -aes256 -in /mnt/sysadmin/passwordsLab/secrets/<name>_encrypted.txt -out normal.txt
    cat normal.txt

You can now read your secret!

### gnupg 


`gnupg` or GNU privacy guard, is another tool that can encrypt
files. In this case, we are going to use a symmetric-key cipher, which
means that both the author and recipient will use the same key for
encryption and decryption. Both parties need access to the public key in
order to send and receive the message, which is the biggest drawback to
the symmetric cipher. To specify a symmetric cipher, add the `-c` option
while encrypting. To encrypt a file, run

    gpg -c --cipher-algo AES256 filename

You will then be asked for a passphrase. After doing this, you'll see
that a file, filename.gpg, has been added to the directory. Now that you
have the encrypted file, you no longer need the old file so feel free to
`rm filename`. If you `cat` the new filename.gpg file, you can see that
the information within it is encrypted.

Unencrypting a file is as easy as encrypting it!

    gpg filename.gpg

You may be asked if you want to overwrite the original, unencrypted file. If you
say "no", then gpg will prompt you to enter another file name. There are plenty
of other types of encryption that you can run. Simply look at the man page for
more details.

## Activity

One of your close friends, Michael, has recently passed away. On his
deathbed, he asked that you retrieve some important data from the file
`lastwords` on his account. Copy his home directory to yours, so that you can
analyze it:

`sudo cp -r /mnt/sysadmin/passwordLab/michael ~/`

**Be sure to copy the directory**, so you don't spoil everyone else's fun if you
modify the files

Now see if you can read his last words!


### Notes

As always, judicious googling is your friend! Here are some other tips
just in case things go wrong:

If you can't get something to work, make sure that you have permissions
for that file or directory.

## Bonus activity

Set up mutt to send encrypted email using gpg public/private keys.

[INCLUDE="encrypted_mail.md"]

## Readings and other references 

 A brief introduction to what encryption is and the various problems
    associated with it for most people:
    http://www.washingtonpost.com/blogs/wonkblog/wp/2013/06/14/nsa-proof-encryption-exists-why-doesnt-anyone-use-it/

More on /etc/passwd: http://www.linfo.org/etc-passwd.html

An introduction to John the Ripper: http://www.admin-magazine.com/Articles/John-the-Ripper

\`\`GnuPG." Gentoo Wiki. N.p., n.d. Web. 15 July 2014.
\<http://wiki.gentoo.org/wiki/GnuPG\>.

\`\`John The Ripper." Juggernaut AppSec Wiki. N.p., n.d. Web. 15 July
2014. \<http://juggernaut.wikidot.com/jtr\>.

\`\`MuttGuide/UseGPG." Mutt. N.p., Dec. 2013. Web. 15 July 2014.
\<http://dev.mutt.org/trac/wiki/MuttGuide/UseGPG\>.

\`\`SSH/OpenSSH/Keys." Ubuntu Documentation. N.p., n.d. Web. 15 July
2014. \<https://help.ubuntu.com/community/SSH/OpenSSH/Keys\>.

\`\`Symmetric Key Algorithm." Wikipedia. Wikimedia Foundation, 06 Nov.
2013. Web. 15 July 2014

\`\`Why AES Is Not Used for Secure Hashing, Instead of SHA-x?" Stack
Exchange. N.p., 12 Oct. 2011. Web. 15 July 2014.
\<http://security.stackexchange.com/questions/8048/why-aes-is-not-used-for-secure-hashing-instead-of-sha-x\>.

\`\`\`\`Password“ Unseated by \`\`123456” on SplashData's Annual
\`\`Worst Passwords“ List.” SplashData, n.d. Web. 14 July 2014.

Ellingwood, Justin. \`\`How To Use PAM to Configure Authentication on an
Ubuntu 12.04 VPS." DigitalOcean. N.p., 3 Oct. 2013. Web. 14 July 2014.

Ellingwood, Justin. \`\`How To Use Passwd and Adduser to Manage
Passwords on a Linux VPS." DigitalOcean. N.p., 17 June 2014. Web. 15
July 2014.

Muffett, Alec. \`\`CrackLib: A ProActive Password Sanity Library." N.p.,
14 Dec. 1997. Web. 14 July 2014.
\<ftp://coast.cs.purdue.edu/pub/tools/unix/libs/cracklib/cracklib.2.7.README\>.

NixCraft. \`\`Linux: HowTo Encrypt And Decrypt Files With A Password."
NixCraft. NixCraft, 8 June 2012. Web. 15 July 2014.
\<http://www.cyberciti.biz/tips/linux-how-to-encrypt-and-decrypt-files-with-a-password.html\>.

Pillai, Sarath. \`\`How Are Passwords Stored in Linux (Understanding
Hashing with Shadow Utils)." Slashroot.in. N.p., 24 Apr. 2013. Web. 15
July 2014.
\<http://www.slashroot.in/how-are-passwords-stored-linux-understanding-hashing-shadow-utils\>.

Srivistava, Vishal. \`\`Understanding and Configuring PAM."
DeveloperWorks. IBM, 10 Mar. 2009. Web. 15 July 2014.

Wilson, Gary, Jr. \`\`Using CrackLib to Require Stronger Passwords."
Gary Wilson Jr. N.p., 26 Aug. 2006. Web. 15 July 2014.

