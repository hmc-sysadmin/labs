INTRODUCTION: ABOUT POSTFIX {#introduction-about-postfix .unnumbered}
===========================

Mail Systems {#mail-systems .unnumbered}
------------

There are three components to mail systems:

1.  **MTA (Mail Transport Agent)**

    MTAs are responsible for getting mail from one place to another.
    This is ALL that they do. Postfix is Knuth’s MTA.

2.  **MDA (Mail Delivery Agent)**

    MDAs take messages and deliver them to the proper mailbox. Knuth
    uses Courier as its MDA.

3.  **MUA (Mail User Agent)**

    This is what the user sees and interacts with. MDAs are responsible
    for reading mails and allowing users to compose mail. In Knuth, we
    use Alpine as our MUA.

Protocols {#protocols .unnumbered}
---------

Protocols define the \`\`language" that systems use when communicating.
This is very useful and efficient because systems don’t have to
translate themselves from one machine to the next when communicating.
Mail runs using SMTP and POP or IMAP.

### SMTP {#smtp .unnumbered}

Postfix is an SMTP, or Simple Mail Transfer Protocol, server. An SMTP
server handles all outgoing mail.

1.  First, the MUA uses port 25 to connect to the server at your email’s
    domain. The MUA then tells the SMTP server the addresses of the
    server and recipient and the body of the message.

2.  The server then breaks down the \`\`to" address into two parts: the
    recipient (part before the @) and the domain name. Since the
    recipient has another domain, SMTP has to communicate with the other
    domain.

3.  To do this, SMTP has a conversation with DNS, the Domain Name
    Server. The DNS will give the domain’s IP address to the SMTP
    server. After this, your SMTP server can connect to the SMTP server
    at the other domain. The other domain will recognize its domain and
    will hand the message to the POP3 or IMAP server.

    Postfix understands very simple commands, which is why we can send
    simple emails in postfix without installing an MUA.

### ESMTP {#esmtp .unnumbered}

ESMTP is an enhanced version of SMTP. It adds greater security and more
features for authentication, securing bandwith, and protecting servers.
Most commercial servers support SMTP as well as ESMTP.

### POP versus IMAP {#pop-versus-imap .unnumbered}

POP, Post Office Protocol, and IMAP, Internet Message Access Procotcol,
are similar in that they both receive and hold emails until the user
comes to pick them up. While they both perform the same job, the go
about it in two completely different ways. Both can be used offline.

### POP {#pop .unnumbered}

POP is older than IMAP, so it has historically been more common. POP
works by contacting your email server through port 110. To access your
text file, the POP server asks for an account name and password.Each
email account has one text file on the server. When a new message comes,
POP will add the new message to the bottom of your respective file. The
new messages will be delivered to your local machine. Once downloaded,
it will usually delete the messages from the server unless you’ve told
the client not to. Because everything is stored on an individual
machine, for those who travel often or access their email from multiple
devices, IMAP is much easier to work with.

### IMAP {#imap .unnumbered}

While POP is older, IMAP is becoming the go-to for email clients. IMAP
contacts the email server through port 143. With IMAP you can read,
search, sort, delete, and organize your mail into folders without
downloading them first. It keeps a record of the messages you send and
also syncs the email that’s on your computer with the email on the
server, so the email always remains on the server. IMAP is usually a lot
quicker than POP because it only downloads messages and attachments when
you click on them. Thus, you can start reading through your email
without waiting for every single message to load.

### Web-based Interfaces {#web-based-interfaces .unnumbered}

The popularity of web-based interfaces like GMail, Yahoo!Mail, and
Outlook.com, fewer people use IMAP and POP. The disadvantage of these
are that the user has to remain connected to the internet at all times
in order to read their mail. In this day and age, this is not a problem
for most people anymore. IMAP and POP are still really cool, and if you
want to create your own email service, they’re the way to go!

TLS CERTIFICATES AND SSL {#tls-certificates-and-ssl .unnumbered}
------------------------

### SSL {#ssl .unnumbered}

Secure Sockets Layer, or SSL, was created by Netscape in the early days
of the internet. SSL creates encrypted connections between your web
server and your visitor’s web server. To enable SSL, you need an SSL
certificate, which is provided by the certificate authorities. Without
one, it is possible for every piece of data that you’re sending to be
seen by others. There are a lot of Certification Authorities and you may
have to pay money for an SSL license.

### TLS {#tls .unnumbered}

TLS, or transport layer security, is the new version of SSL. While you
can choose to pay for a license if you so wish, you can also get a
license for free. Fun fact: when you see https, it means that two
clients are exchanging http over a secure SSL/TSL connection.

More information {#more-information .unnumbered}
----------------

If you are curious and want to learn more about Postfix, these are good
resources:

http:/www.linuxjournal.com/article/9454?page=0,0

http:/www.postfix.org/OVERVIEW.html

POSTFIX {#postfix .unnumbered}
=======

In this lab, we will set up our own mail server and try to make it
secure. We will be using Postfix as our MTA, and mutt as our MUA. Both
postfix and mutt should already be installed on your machine.

Troubleshooting Hints {#troubleshooting-hints .unnumbered}
---------------------

If you come across errors, maybe emails don’t send or you get a bounce
message, the first thing you should do is check the mail log file,
`/var/log/mail.log`. The most recent emails are at the bottom of the
file (Reminder: SHIFT-G will take you to the bottom of a file in vim).
Errors are easy to spot as they are usually highlighted in red.\

Often times the errors are related to file ownership and permissions.
Make sure that the account that needs to access a file (whether the root
account or your account) owns the file, and check that the permissions
make sense. Also, as usual, google is your friend (effective googling is
a good skill to learn!). This may sound silly, but errors are often the
result of a simple misspelling or a missing word. Double check that
everything is typed correctly before looking for configuration
problems.\

Happy Postfix configuring and email sending!

Set Up {#set-up .unnumbered}
------

1.  Double-check that the hostname of your computer is set correctly. In
    the file /etc/conf.d/hostname, there should be a line that defines
    the hostname of your computer (example: `hostname="HOST_NAME"`).
    Note: This step should have already been completed in the Gentoo
    install.

2.  You can change various settings and options for your mail system in
    the file `/etc/postfix/main.cf`. You should uncomment the line
    `$my_destination` that contains\
    `$myhostname, localhost.$mydomain,localhost, $mydomain`. You are
    welcome to change settings as necessary or just for fun. Go ahead
    and play around with it!

3.  The next required step is to create the alias database by running
    the command `newaliases`. This needs to be run every time settings
    in postfix are updated.

        newaliases

4.  Now you’re ready to start postfix! Simply run the command below. If
    everything went according to plan, you will now be able to send
    messages.

        /etc/init.d/postfix start

    The syntax of a message is below:

        sendmail EMAIL_ADDRESS
            (FROM: _____)
            (TO: _____)
            (CC: ______)
            (SUBJECT: _____)
            message text
            .

    The lines in parentheses (from, to, cc, and subject) are optional
    and just change how the message looks when it is received. Note that
    you may need to use sudo to send emails. Avoid extra lines between
    lines because postfix gets confused.

    Now add postfix to the runlevel so that it will start when your
    computer boots up.

        sudo rc-update add postfix default

Your first task is to send an email to yourself (your Mudd email or
other email account)! Set the email to be from your favorite superhero
and to your favorite supervillain (may or may not be related). Make sure
you include a subject line and put text in the message. When checking
that you received the email, you might need to look for it in your spam
folder (more on spam later).

CHANGE HOSTNAME AND DOMAIN NAME {#change-hostname-and-domain-name .unnumbered}
===============================

The name before the @ sign is defined using the \`\`From:" line in
`sendmail`. Your goal now is to change the bit that appears after the @
sign in your email address. Do this by editing the main.cf file.
Configure the file so that your address looks like
@HOSTNAME.sys.cs.hmc.edu. (Hint: remember you can search documents in
vim for a keyword using `/keyword`).\

In order for changes to the main.cf file to take effect, run the command
below. You will need to do this anytime you make changes to main.cf.

    postfix reload

HOW DOES MAIL WORK? {#how-does-mail-work .unnumbered}
===================

You can get a brief overview of how this process works by using
`telnet`. Mail sends using port 25, so you must establish a connection
specifying that port. First connect to your own machine.

    telnet hostname.domainname 25

You will then get a reply that looks like this:

    Trying 127.0.0.1...
    Connected to hostname.domainname.
    220 hostname.domainname ESMTP Postfix

Then run:

    HELO hostname.domainname 

Anything that starts with a \`\`2“ is a good sign! A \`\`3” means the
server is waiting for you to send the text of an email and \`\`4“ and
\`\`5” mean that the server is unhappy with you. Once connected, the
server sends you its \`\`banner", which is the part after 220. To get
more information about the machine you just connected to, run
`EHLO hostname.domainname`. Once you have done this, tell the server who
the mail is from.

    MAIL FROM: yo_son@hostname.domainname

You will get a response from the server, saying everything is OK. Then
you have to specify who you’re sending to (send to a valid mail address,
like your gmail account):

    RCPT TO: yo_mama@mamas.org

Again you will receive a response from the server saying everything is
OK. After that, type in `DATA`. The server will again confirm that it
has received your message. You can then input your message contents as
you did using the `sendmail` command.

    From: yo son
    To: yo mama
    Subject: yo mama so stupid she sold her car fo gas money

    jk i luv you mama <3         
    .

You will receive another confirmation message, after which you can
`QUIT` telnet. You can also check your email through telnet by using
port 110, but we’ll leave that for you to explore in your free time.
Sendmail makes you go through these same steps, but it’s a lot easier to
send a mail through that than through telnet. Telnet is also much more
unsecure than postfix. Let’s just say our generation has it good.

MUTT SETUP {#mutt-setup .unnumbered}
==========

Mudd’s CS department uses alpine as its MUA (Mail User Agent), but mutt
is easier to install simply because it has a nicely fleshed-out Gentoo
wiki page, and so your machine is running mutt. Also, mutt was made by a
Mudd alum!

1.  To start mutt, simply type mutt into the command line:

        mutt

    If Mutt gives you the error that .maildir is not a maibox, that is
    an easy fix. Go to the .maildir directory within your home directory
    and create three directories: `cur`, `new`, and `tmp`. This should
    allow Mutt to recognize the .maildir as a mailbox.

2.  In order for new emails to be sent to the mutt program, you need to
    specify the mailbox. Emails need to be sent to the directory that
    mutt recognizes as its mailbox. Open the file `/etc/postfix/main.cf`
    and search for `home_mailbox`. Below the suggested lines for
    defining this variable, add the following:

        home_mailbox = .maildir/

Congrats and enjoy your MUA!\

Use mutt to send an email to yourself. Compare sending an email with
postfix using the sendmail command in the command line and sending an
email through mutt.

ALIASES {#aliases .unnumbered}
=======

The next step is to set up aliases, or mailing lists. This allows you to
add emails to a list and send a message to that list. The CS department
uses TONS of aliases for various clinic and research groups as well as
courses and staff/faculty lists.\

In the directory `/etc/mail`, there should be a file called `aliases`.
If not, go ahead and create it. Make sure you have at least these two
aliases: `mailer-daemon:postmaster` and `postmaster:root`. These are
aliases that are required by the system. Next, go to the file
/etc/postfix/main.cf. and find \`\`Alias Database". Uncomment the
correct `alias_maps` line, the one that contains `hash` and the correct
file pathway (/etc/mail/aliases). If none of the lines match exactly, go
ahead and write your own. Also uncomment the correct `alias_database`
line with the same file pathway. `hash` means that postfix will create
and reference a database (.db) file. You do not need to create the .db
file, postfix will automatically generate it for you!\

Set up a mail alias and add your email and your friend’s email to it.
You may need to use sendmail from the command line and not Mutt for to
send an email to an alias. The syntax is:
`alias_name:email_1,email_2,...`. In order for the change to take
effect, you must run the command `newaliases` as root. If you ever want
to reference multiple alias files, you can add the additional files to
the correct lines in main.cf (`alias_maps` and `alias_database`). Don’t
forget to reference them with `hash` since they are database files.

SCAVENGER HUNT! {#scavenger-hunt .unnumbered}
===============

Your final task is to go on a quest! The first step is to send an email
to bacon@crispy.sys.cs.hmc.edu and follow the instructions from there
:)\

Extra Credit! {#extra-credit .unnumbered}
=============

Personalization {#personalization .unnumbered}
---------------

Look up a sample .muttrc configuration file and personalize your inbox.
Options you may want to set are `realname`, to change the name that
appears when you send email to someone; `editor`, to change the text
editor used to write emails from nano to something else (like vim); and
`alias_file`, to allow you set shorthand to send emails faster (for
example, set `alias NAME EMAIL_ADDRESS` so you can simply send an email
to NAME without needing to type the entire address).\

You can also set colors of text, like the From or Subject line of an
email, error messages, or the color of the email that’s highlighted.
Feel free to personalize as much as you want!

Vacation {#vacation .unnumbered}
--------

If you have extra time, you can try setting up your own auto-reply
email. This can be done through a postfix add-on program called
`vacation`. To begin your `vacation`, start by running

    vacation

This will create a `.vacation.msg` file in your home directory. Edit it
as you so wish. The last thing you need to do is look for a .forward
file in your home directory. The .forward file should only contain a
single line of code:

    \username, "|/usr/bin/vacation username"

This will ensure that one copy of every incoming message will be sent to
`username` and another copy will be sent to the vacation program. After
you do this, run

    vacation -I

This will create a `.vacation.db` file. Make sure both the
`.vacation.msg` and the `.vacation.db` are owned by their respective
user. Otherwise, the vacation file will not work as desired.

As always, if something isn’t working make sure to check your mail log
files to see what’s wrong.

SpamAssassin {#spamassassin .unnumbered}
------------

SpamAssassin is a thing you can install if you’re courageous.

