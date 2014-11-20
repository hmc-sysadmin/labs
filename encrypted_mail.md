### Encrypted Mail

I bet you've always wanted to send encrypted emails. Well, this is the
chance to live your dreams!

It's pretty easy to set up `gpg` on `mutt` once you have your own `gpg`
public key, but the process of getting this key can be a bit tricky.

    gpg --gen-key

You will then be prompted to to select what kind of key you want. Go
ahead and select the default, `RSA and RSA`. Then select 2048 as your
keysize and choose how long you want your key to last. After that,
you'll need to input some user information. You can use real information
or fake–this just makes it easier to locate your key. You'll be asked to
enter in a passphrase. Make sure you remember what this password is. If
you lose it, you can't get it back! Once you've filled out all the
necessary information, GnuPG will generate your key. This might take a
while. You can speed the process up by keyboard mashing. `gpg` generates
your new key by using the time interval between keystrokes.

You can also log into another session and run the following command, to generate
some disk activity:

    sudo dd if=/dev/sda of=/dev/zero

Once your key is generated, you can run the command:

    gpg --list-keys

You should see your new key within your list of keys. For example:

```
pub   2048R/F95D97F2 2014-11-04 [expires: 2014-12-14]
uid       [ultimate] Ben Wiedermann <ben@adminvm5.sys.cs.hmc.edu>
sub   2048R/A4F464B6 2014-11-04 [expires: 2014-12-14]
```

You'll see that there is the `pub`lic key and the `sub`key of the `pub` key.
Usually people use the `pub` key for signing and the `sub` key for encryption.

When you need to use your key for other commands (e.g., the commands below), you
want the value after the slash for the `pub` key , e.g., `F95D97F2`.

Before doing anything else, you should create a revocation key. This
allows you to revoke your key in the event of a disaster.

    gpg --output revoke.asc --gen-revoke <your key>

Select "1“ and "yes".

Now you're ready to integrate `gnupg` into `mutt`. Go into your sample
`mutt` configuration files. `mutt` comes pre-installed with a `gnupg`
config file within the sample directory.

    cd /usr/share/doc/mutt-*/samples

Inside this directory, you should see a `gpg.rc` file. Copy this file
into your home directory so that it'll work for your `mutt`.

    cd /usr/share/doc/mutt-*/samples/gpg.rc ~/.gpgrc

If there is a .bz2 extension after it, you won't be able to copy it to
your home directory until you unzip it.

    bzip2 -d gpg.rc.bz2

Add the following lines to the `.muttrc` file in your home directory:

    source = ~./gpgrc
    set pgp_use_gpg_agent = yes
    set pgp_sign_as = <yourkey> 
    set pgp_timeout = 3600
    set crypt_replyencrypt = yes

#### Troubleshooting
It can take a while to generate enough entropy to create an GnuPG key.
Be patient and if it doesn't work, try making another one or consulting
the internet.

### Find a Friend 

Look around you and find a buddy to send an email to.

The people you're sending mail to need to have your key or they won't be
able to read the mail they receive from you. There are two ways to do
this: creating an ASCII version of your key or uploading your key onto a
keyserver.

1.  ASCII Version

        gpg --armor --output public.key --export <Key ID>

    `public.key` can be whatever name you want. It's helpful to have it
    be something that's easily identifiable.

    Once that's done, put your key in the NFS-mounted directory 
    `/mnt/local/keys`. You can also email your key to your friend
    Once you've done that, all they have to
    do is import it:

        gpg --import <public.key>

2.  Keyservers

    Alternatively, to make it easier to find your key, upload your key
    onto the interwebs.

        gpg --keyserver subkeys.pgp.net --send-keys <Key ID>

    To have your friend get your key, they just have to do the reverse
    process, where "thriller@gmail.com" is the email that you listed
    in your key's UID.

        gpg --keyserver subkeys.pgp.net --search-keys thriller@gmail.com

    Once your friend has found your key, they just have to import it
    into your keyring.

        gpg --import <friend's key ID>

Once that's done, and you've tried to import your friend's key, check to
see if their key is in your keyring.

    gpg --list-keys

If it's there, then everything is perfect! Open mutt and compose a mail
to your buddy. Then, on the last screen, instead of pressing "y“ to
send your mail, press "p". You will then get a couple of options for
what to do with your mail. Choose "b"–both sign and encrypt. You will
be prompted for your password and for your friend's key. Enter them and
everything should go smoothly. Now you're sending encrypted mail!