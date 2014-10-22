# Email, with postfix and mutt

*Originally created by Lisa Goeller and Paige Rinnert*

## Introduction

In this lab, we will set up our own mail system and try to make it
more secure. 

As usual, the sysadmin code applies: **do not use what you learn in this lab to
send spam or forge emails, unless you have permission to do so as part of the
lab (and then, only to authorized systems)**.

## Overview: Mail Systems

We'll focus on two major components to mail systems:

**Mail Transport Agent (MTA)**. MTAs are responsible for getting mail from
one place to another, typically among different machines. We'll use a program 
called Postfix to transport mail.

**Mail User Agent (MUA)**. MUAs are what users see and interact with, to
read and compose mail. We'll use a program called Mutt to read and compose mail.

## Overview: Protocols 

Protocols define the "language" that systems use when communicating.
This is very useful and efficient because systems don't have to
translate themselves from one machine to the next when communicating.
Mail runs using SMTP (for sending) and POP or IMAP (for retrieving).

### SMTP 

Postfix is an SMTP, or Simple Mail Transfer Protocol, server. An SMTP
server handles all outgoing mail.

1.  First, the MUA uses port 25 to connect to the server at your email's
    domain. The MUA then tells the SMTP server the addresses of the
    server and recipient and the body of the message.

2.  The server then breaks down the "to" address into two parts: the
    recipient (part before the @) and the domain name. Since the
    recipient has another domain, SMTP has to communicate with the other
    domain.

3.  To do this, SMTP has a conversation with DNS, the Domain Name
    Server. The DNS will give the domain's IP address to the SMTP
    server. After this, your SMTP server can connect to the SMTP server
    at the other domain. The other domain will recognize its domain and
    will hand the message to the POP3 or IMAP server.

    Postfix understands very simple commands, which is why we can send
    simple emails in postfix without installing an MUA.

### POP versus IMAP 

POP, Post Office Protocol, and IMAP, Internet Message Access Procotcol,
are similar in that they both receive and hold emails until the user
comes to pick them up. While they both perform the same job, the go
about it in two completely different ways. Both can be used offline.

### POP 

POP is older than IMAP, so it has historically been more common. POP
works by contacting your email server through port 110. To access your
text file, the POP server asks for an account name and password. Each
email account has one text file on the server. When a new message comes,
POP will add the new message to the bottom of your respective file. The
new messages will be delivered to your local machine. Once downloaded,
it will usually delete the messages from the server unless you've told
the client not to. Because everything is stored on an individual
machine, for those who travel often or access their email from multiple
devices, IMAP is much easier to work with.

### IMAP 

While POP is older, IMAP is becoming the go-to for email clients. IMAP
contacts the email server through port 143. With IMAP you can read,
search, sort, delete, and organize your mail into folders without
downloading them first. It keeps a record of the messages you send and
also syncs the email that's on your computer with the email on the
server, so the email always remains on the server. IMAP is usually a lot
quicker than POP because it only downloads messages and attachments when
you click on them. Thus, you can start reading through your email
without waiting for every single message to load.

## Secure Sockets Layer (SSL) and Transport Layer Security (TLS) certificates

### SSL 

Secure Sockets Layer, or SSL, was created by Netscape in the early days
of the internet. SSL creates encrypted connections between your web
server and your visitor's web server. To enable SSL, you need an SSL
certificate, which is provided by the certificate authorities. Without
one, it is possible for every piece of data that you're sending to be
seen by others. There are a lot of Certification Authorities and you may
have to pay money for an SSL license.

### TLS 

TLS, or transport layer security, is the new version of SSL. While you
can choose to pay for a license if you so wish, you can also get a
license for free. Fun fact: when you see https, it means that two
clients are exchanging http over a secure SSL/TSL connection.

## Sending and receiving mail: troubleshooting hints 

As you go through the lab, you may across errors: maybe emails don't send or you
get a bounce message. The first thing you should do is check the mail log file,
`/var/log/mail.log`. The most recent emails are at the bottom of the file
(Reminder: SHIFT-G will take you to the bottom of a file in vim). Errors are
easy to spot as they are usually highlighted in red.

Often times the errors are related to file ownership and permissions.
Make sure that the account that needs to access a file (whether the root
account or your account) owns the file, and check that the permissions
make sense. Also, as usual, google is your friend (effective googling is
a good skill to learn!). This may sound silly, but errors are often the
result of a simple misspelling or a missing word. Double check that
everything is typed correctly before looking for configuration
problems.

## Sending mail

First, we'll set up postfix, the transfer agent responsible for sending mail
among computers.

Although many of the commands in this lab require root, you should **log on to
your machine using your user name (not root )**. Use `sudo` to execute the
commands.

### Configure and start postfix

1.  You can change various settings and options for your mail system in
    the file `/etc/postfix/main.cf`. You should uncomment the line
    `$my_destination` that contains

        $myhostname, localhost.$mydomain,localhost, $mydomain

1.  The next required step is to create the alias database by running
    the command `newaliases`. This needs to be run every time aliases are 
    updated.

        sudo newaliases

1.  Now you're ready to start postfix. Simply run the command below.

        sudo /etc/init.d/postfix start

1.  You should also configure your system so that postfix will start when your
    computer boots up.

        sudo rc-update add postfix default

You're ready to start sending mail! Using the command below, send an email to
yourself (your school email or other email account). Make sure you include a 
subject line and put text in the message. When checking that you received the 
email, you might need to look for it in your spam folder.

The command to send a message is:

        sendmail EMAIL_ADDRESS
            (FROM: _____)
            (TO: _____)
            (CC: ______)
            (SUBJECT: _____)
            message text
            .

The lines in parentheses (from, to, cc, and subject) are optional. 
Avoid blank lines because they confuse postfix.

### Change the settings for hostname and domain name
Did you get your email? The sending address should have the form:
    from@hostname
The email address has two parts:

  - **from**: by default, the username on the account that sent the email. It 
  can also be defined using the "From:" line in `sendmail`
  - **hostname**: defined using the `myhostname` setting in /etc/postfix/main.cf.
  If this setting is undefined, postfix will pull it from the settings in
  /etc/conf.d/hostname

Your goal now is to change the part that comes after the @ sign. Do this by
editing the configuration file, /etc/postfix/main.cf. Configure the file so that
your address looks like @VMNAME.sys.cs.hmc.edu.

In order for changes to the main.cf file to take effect, run the
command below. You will need to do this anytime you make changes to main.cf.

    postfix reload

### Aside: How does sending mail work? 

You can get a brief overview of how the mail transfer agent works by using
`telnet`. Mail sends using port 25, so you must establish a connection
specifying that port. First connect to your own machine.

    telnet hostname.domainname 25

You will then get a reply that looks like this:

    Trying 127.0.0.1...
    Connected to hostname.domainname.
    220 hostname.domainname ESMTP Postfix

Then run:

    HELO hostname.domainname 

Anything that starts with a "2“ is a good sign! A "3” means the
server is waiting for you to send the text of an email and "4“ and
"5” mean that the server is unhappy with you. Once connected, the
server sends you its "banner", which is the part after 220. To get
more information about the machine you just connected to, run
`EHLO hostname.domainname`. Once you have done this, tell the server who
the mail is from.

    MAIL FROM: me@hostname.domainname

You will get a response from the server, saying everything is OK. Then
you have to specify who you're sending to (send to a valid mail address,
like your gmail account):

    RCPT TO: you@yourplace.org

Again you will receive a response from the server saying everything is
OK. After that, type in `DATA`. The server will again confirm that it
has received your message. You can then input your message contents as
you did using the `sendmail` command.

    From: me
    To: you
    Subject: An exciting investment opportunity

    jk, but your mail client probably thinks this is spam
    .

You will receive another confirmation message, after which you can
`QUIT` telnet. You can also check your email through telnet by using
port 110, but we'll leave that for you to explore in your free time.
Sendmail makes you go through these same steps, but it's a lot easier to
send a mail through that than through telnet. Telnet is also much less
secure than postfix. Let's just say our generation has it good.

## Receiving mail

Mudd's CS department uses alpine as its MUA (Mail User Agent), but your machine
will use mutt. Mutt is easier to install simply because it has a nicely 
fleshed-out Gentoo wiki page, and so your machine is running mutt. Also, mutt
was made by a Mudd alum (Michael Elkins, '93)!

1.  To start mutt, simply type mutt into the command line:

        mutt

    Mutt will offer to create a mailbox for you. You may also get 
    the error that .maildir is not a maibox, go to the .maildir directory 
    within your home directory and create three directories: `cur`, `new`, 
    and `tmp`. This should allow Mutt to recognize the .maildir as a mailbox.

2.  In order for new emails to be sent to the mutt program, you need to
    specify the mailbox. Emails need to be sent to the directory that
    mutt recognizes as its mailbox. Open the file `/etc/postfix/main.cf`
    and search for `home_mailbox`. Below the suggested lines for
    defining this variable, add the following:

        home_mailbox = .maildir/

3. Remember to reload your postfix settings!

Use mutt to send an email to your school email address. Compare sending an email
with postfix using the sendmail command in the command line and sending an email
through mutt. In particular, what does the `from` field look like in the email
you sent from mutt? If it doesn't match the one from sendmail, you may need to
configure mutt to have the correct `from` field. To do so, create a file in your
home directory called `.muttrc` and add the following line to the file:
```
set from="<user>@<machine-name>.sys.cs.hmc.edu"
```

## Aliases 

The next step is to set up aliases, or mailing lists. This allows you to
add emails to a list and send a message to that list. The CS department
uses TONS of aliases for various clinic and research groups as well as
courses and staff/faculty lists.

In the directory `/etc/mail`, there should be a file called `aliases`.
If not, go ahead and create it. Make sure you have at least these two
aliases: `MAILER-DAEMON:postmaster` and `postmaster:root`. These are
aliases that are required by the system. 

Next, go to the file
/etc/postfix/main.cf. and find "Alias Database". Uncomment the
line for `alias_maps` that contains `hash` and the correct
file (/etc/mail/aliases). If none of the lines match exactly, go
ahead and write your own. Also uncomment the correct `alias_database`
line with the same file. `hash` means that postfix will create
and reference a database (.db) file. You do not need to create the .db
file, postfix will automatically generate it for you!

Edit `/etc/mail/aliases` to set up a mail alias and add your email to it.
You may need to use sendmail from the command line and not Mutt to
send an email to an alias. The syntax is:
`alias_name:email_1,email_2,...`. In order for the change to take
effect, you must run the command `newaliases` as root. If you ever want
to reference multiple alias files, you can add the additional files to
the correct lines in main.cf (`alias_maps` and `alias_database`). Don't
forget to reference them with `hash` since they are database files.

## Scavenger Hunt! 

Your final task is to go on a quest! The first step is to send an email
to bacon@crispy.sys.cs.hmc.edu and follow the instructions from there
:)

## Bonus: Personalization 

Look up a sample .muttrc configuration file and personalize your inbox.
Options you may want to set are `realname`, to change the name that
appears when you send email to someone; `editor`, to change the text
editor used to write emails from nano to something else (like vim); and
`alias_file`, to allow you set shorthand to send emails faster (for
example, set `alias NAME EMAIL_ADDRESS` so you can simply send an email
to NAME without needing to type the entire address).

You can also set colors of text, like the From or Subject line of an
email, error messages, or the color of the email that's highlighted.
Feel free to personalize as much as you want!

## Bonus: Vacation 

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

As always, if something isn't working make sure to check your mail log
files to see what's wrong.

## Bonus: SpamAssassin 

SpamAssassin is a thing you can install if you're courageous.

## More information 

If you are curious and want to learn more about Postfix, these are good
resources:

http:/www.linuxjournal.com/article/9454?page=0,0

http:/www.postfix.org/OVERVIEW.html



