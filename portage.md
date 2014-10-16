# Package Management with Portage

*Originally created by Huting Lin*

## Overview

Portage is Gentoo's package management system. It's what you use to install
and manage programs, libraries, etc. on Gentoo systems. In general, portage uses
the term *package* to refer to something that can be installed on Gentoo. Most
packages have *dependencies*, i.e., other packages whose installations are pre-
requisites for installing the desired package. Portage, and in particular, the
`emerge` command manages dependencies for the packages installed on the system.

## Using emerge to calculate dependencies for a package

emerge is one of the most important commands in relation to Portage.
The command allows users to install packages, calculate dependencies, and more.

The emerge command has an option called `pretend` that allows the user to see
what the installation would do if it *were* to install a package, but it doesn't
actually install the package.

As an example, let's pretend to install the *cowsay* package. The *cowsay*
package is a program that takes an input string and prints out ASCII art of a
cow saying (or thinking) the string. Run the following command to pretend to
install the package:

```
emerge --pretend cowsay
```

The emerge program will describe all the dependencies of cowsay, along with
each dependency's status (i.e., what emerge plans to do with each dependency).
Here are the possible status values and their meanings:

N
:   package is completely new, so installing new package

S
:   new SLOT installation (can be installed side-by-side with other
    versions) *discussed in more detail below*

U
:   updating to another version

D
:   downgrading to lower version since it seems better than current
    version

r
:   reinstall (forced)

R
:   replacing/remerging same version

F
:   fetch restricted; must be manually downloaded

f
:   fetch restricted; already downloaded

I
:   interactive/requires user input

B
:   blocked by another package (unresolved conflict) 
    *discussed in more detail below*

b
:   blocked by another package (automatically resolved conflict)

## Installing a package

Now install the package for real, by running:
```
sudo emerge --ask cowsay
```

The `ask` option lets you review what emerge plans to do, then consent. The
program will download, configure, compile, and install *cowsay* and all its
dependencies.

Now that *cowsay* is installed, you can run the program like so:
```
cowsay hello world
```

## Removing a package

To remove a package, we should first check how the removal would affect our
system. We might think twice about removing a package on which lots of other
things depend! 

For example, let's see what would happen if we were to remove *python*:
```
emerge --pretend --verbose --depclean python
```

The `depclean` option asks emerge to remove the specified package, along with
all dependencies of the package on which no other package depends.

You'll notice from the output that other packages depend on *python*, so emerge
refuses to remove python. (There are ways to force removal, but that's generally
a bad idea---it can lead to a broken system.)

NOTE: A command's options usually have shortened versions to make things easier
on the user. The shortened version is often (though not always) the first letter
of the option. So, `--pretend` can be shortened to `-p`,  `--ask` can be
shortened to `-a`, and `depclean` can be shortened with `-c`. Options can also
be combined without spaces using their shortened form. So, if we wanted, 
`--depclean`, `--ask`, and `--verbose`, then we can use `-cav`. 

  1. Remove *cowsay*, using the ask option so that you can review emerge's
plans and consent to them.

  1. Verify that *cowsay* is gone by trying to run the command.

  1. Reinstall *cowsay*.

  1. Verify that *cowsay* is back by trying to run the command.

## More on dependencies

A dependency is a file that the package you are trying to install
requires. When a package is emerged, it usually "pulls in" the
dependencies needed in the installation. It draws on dependency trees
(lists/records of each package and what it depends on or what depends on
it) synced from Gentoo, and can encounter problems with conflicting
configurations or dependencies (more on this later).

### Display Dependencies

Portage has dependency trees for its packages to keep track of what is
required by the package or what it requires. 

Let's explore the dependencies for the *fortune* package. The *fortune* package
provides a command that prints a pithy quote to the terminal. The `fortune`
command can pull its quotes from multiple *fortune files*, and there are lots of
fortune files available. Let's see:
```
equery depgraph fortune-mod-all | less
```

Check out all those dependencies! Each one is a fortune file.

  - How would you find the list of packages that depend on *perl*? 
  (Hint: man pages are your friends!)
  - Install the *fortune-mod-all* package.

Because *fortune-mod-all* has so many dependencies, you might want to do a
little reading while it's installing... All the subsequent sections are bonus:
more information that may help you when administering your system. Feel free to
read through them or read up more on portage online.

When *fortune-mod-all* is done installing, let's have some fun:

   - Run `fortune` to get a pithy message
   - Run `fortune hitchhiker` to get another pithy message
   - Make a cow say a fortune
   - When you start a new bash shell (including when you log in), bash treats 
   the file `~/.bashrc` as a script and runs it. Modify `~/.bashrc` so that a
   cow says a pithy quote every time you log in (or start a new shell).
   - You can also make it so that a cow says a pithy quote when *any* user logs
   on to your system. To do so, add the appropriate command to the bottom of the
   file `/etc/profile`. Ask a friend who has an account on your system to log in
   and verify that they're greeted by a cow.

## Updating the Portage Tree and Your System

**WARNING: DO NOT RUN ANY OF THE COMMANDS UNLESS YOU HAVE *LOTS* OF TIME (and
even then, run them with caution)**

The Portage tree is a collection of ebuilds, which are files that
contain all information Portage needs to maintain software (install,
search, query, ...). These ebuilds reside in `/usr/portage` by default
and are the ones that Portage uses when asked to perform some action
regarding software titles.

1.  To make sure that the Portage Tree on your system is up-to-date, you
    would connect to the Gentoo database using:

        emerge --sync

    This will make sure that your Portage Tree matches the one Gentoo
    has on file.

2.  The previous step only updates the tree. No other changes are made
    to the system. To update all the packages on the system, we would use the
    following command. The most important options are `--update` and
    `@world`:

        emerge --ask --update --deep --with-bdeps=y --newuse @world

    -   `--ask` This option means that, after gathering all the data it
        needs and displaying what it will do, `emerge` will ask for user
        confirmation before continuing the process.

    -   `--update` This is the option that tells `emerge` to go through
        the update process. It updates packages to the best version
        available, not necessarily the highest version due to packages
        possibly in the testing stage.

    -   `--deep` This forces Portage to consider the entire dependency
        tree of packages (more information on this topic later in the
        lab) rather than focusing on their more immediate dependencies.

    -   `--with-bdeps=y` When calculating dependencies, this option
        tells `emerge` to include build time dependencies, which are
        dependencies that some packages on your system need during their
        compile and build process. But once that package is installed,
        these dependencies are no longer required

    -   `--newuse` tells `emerge` to consider installed packages when
        USE flags have changed. This helps update already installed
        packages to change accordingly to those changes in USE flags (more on
        USE flags, below).

    -   `@world` tells the command to work at the world level, which
        encompasses the system set and the selected set. The system set
        contains the software packages that are required for a standard
        Gentoo Linux installation to run properly. The selected set
        contains the packages the user has explicitly asked to be
        installed.

3.  To remove (recently) orphaned dependencies (packages that no are no
    longer depended on by another package), you can use the option
    `--depclean` with `emerge`, which cleans the system by removing
    packages that are not associated with explicitly merged packages.

4.  To properly finish off updates/removal of orphan dependencies and to
    make sure all merged packages can properly deal with the change in
    dependencies, you can use the following command. However, it is part
    of a package called `gentoolkit`, so you would need to
    `emerge gentoolkit` before you can use it.

        revdep-rebuild

5.  If you ever get a message to update your config files, there are three
    possible commands to use:

    `etc-update`
    :   is a tool to help merge changes made in config files during
     updates, it gives users the option to keep the old config file,
     use the new one, make changes interactively, or to ignore the
     update. Once in the interactive interface, it gives you a list
     of commands and what each command does.

    `dispatch-conf`
    :   allows the user to utilize an interactive process similar to
    that of **etc-update** along with automatic backups.

    `cfg-update`
    :   is an easy-to-use script with a smart auto-update function (but
    without the risk of breaking your system) and automatic backups.
    It also has the potential to use GUI diff/merge tools for
    updating all configuration files that contain your customized
    settings.

### Possible Problems

As mentioned before, some problems can come up due to conflicts between
dependencies or a dependency and configurations. When problems do arise, you'll
need more information than is in this document, but here's a good start.

-   **Slots**
    Portage allows for different versions of a single package to coexist
    on a system. While other distributions tend to name their package
    those versions (like `python2.6` and `python2.7`), Portage uses a
    technology called `SLOTs`. An ebuild declares a certain `SLOT` for
    its version. Ebuilds with different `SLOTs` can coexist on the same
    system. For instance, the freetype package has ebuilds with
    `SLOT="1"` and `SLOT="2"`. To list all versions of python currently
    installed on your machine, you can use the command:

        eselect python list

    Not all packages have different `SLOTs` for their versions though,
    so if a package tries to install another version, the new version
    would compete with the pre-existing one and result in what is called
    a `slot conflict`. This can be solved by `unmerging` the previous
    version (but ONLY if no other installed package depends on it) or
    mask the new version to prevent `emerge` from pulling it in (see
    "Masked Packages" for more info).

-   **Blocked Packages**
    Ebuilds contain specific fields that inform Portage about its
    dependencies, two of which are `DEPEND` and `RDEPEND`. The two
    possible dependencies, build dependencies and run-time dependencies,
    are declared in these two fields respectively. When one of these
    dependencies explicitly marks a package as "not compatible," it
    triggers a blockage. While recent versions of Portage are smart
    enough to work around minor blockages without user intervention,
    occasionally you will need to fix it yourself.
    
    As an example, pretend that, while trying to emerge the mail
    system Postfix, your `emerge --pretend` gives you the output:

        mail-mta/ssmtp 
        (is blocking mail-mta/postfix-2.2.2-r1)

    and without the option `--pretend`, you got:

        !!! Error: the mail-mta/postfix package conflicts with 
        !!! another package.
        !!!        both can't be installed on the same system together.
        !!!        Please use 'emerge --pretend' to determine blockers. 

    To fix a blockage, you can choose not to install the package or
    unmerge the conflicting package first. In the given example, you can
    opt not to install *postfix* or to remove *ssmtp* first.\
    NOTE: You may also see blocking packages with specific atoms, such
    as\
    `<media-video/mplayer-1.0_rc1-r2` (see `man emerge` for more
    information on atoms). The less-than symbol in the beginning of the
    atom name indicates the group of versions of this partucular
    software that the blockage refers to. So, the "\<" shows that any
    version less than `1.0_rc1_r2` is blocked. So, for this example,
    updating to a more recent version of the blocking package would
    circumnavigate and remove the block.

    It is also possible that two packages that are yet to be installed
    are blocking each other. In this rare case, you should find out why
    you need to install both. In most cases you can do with one of the
    packages alone. If not, you should file a bug to Gentoo.

-   **Masked Packages**

    When you want to install a package that hasn't been marked available
    or compatible for your system, you will receive a masking error
    similar to that of the following:

        !!! all ebuilds that could satisfy "bootsplash" have been masked. 

    with the reason(s) being:

        !!! possible candidates are:

        - gnome-base/gnome-2.8.0_pre1 (masked by: ~x86 keyword)
        - lm-sensors/lm-sensors-2.8.7 (masked by: -sparc keyword)
        - sys-libs/glibc-2.3.4.20040808 (masked by: -* keyword)
        - dev-util/cvsd-1.0.2 (masked by: missing keyword)
        - games-fps/unreal-tournament-451 (masked by: package.mask)
        - sys-libs/glibc-2.3.2-r11 (masked by: profile)
        - net-im/skype-2.1.0.81 (masked by: skype-eula license(s)) 

    You can install a different yet similar application that is
    available for your particular system or wait until the wanted
    package becomes available. Reasons for a package being masked
    include:

    -   `~arch keyword` means that the application is not tested
        sufficiently to be put in the stable branch. Wait a few days or
        weeks and try again.

    -   `-arch keyword` or `-* keyword` means that the application does
        not work on your architecture. An architecture is a family of
        CPUs (processors) who support the same instructions. The two
        most prominent architectures in the desktop world are the x86
        architecture and the x86\_64 architecture (for which Gentoo uses
        the amd64 notation). If you believe the package does work file a

        bug at Gentoo's bugzilla website.

    -   `missing keyword` means that the application has not been tested
        on your architecture yet. Ask the Gentoo architecture porting
        team to test the package or test it for them and report your
        findings on the bugzilla website.

    -   `package.mask` means that the package has been found corrupt,
        unstable, or worse and has been deliberately marked as
        do-not-use.

    -   `profile` means that the package has been found not suitable for
        your profile. The application might break your system if you
        installed it or is just not compatible with the profile you
        use.\
        A Portage profile specifies default values for global and
        per-package USE flags, specifies default values for most
        variables found in /etc/portage/make.conf, and defines a set of
        system packages, basically acting as the default configurations
        of the system. They are maintained by Gentoo developers as a
        part of the Portage Tree.

    -   `license` means that the package's license is not compatible
        with your ACCEPT\_LICENSE setting. You must explicitly permit
        its license or license group by setting it in
        `/etc/portage/make.conf` or in\
        `/etc/portage/package.license`. Refer to **Licenses** in the
        Gentoo Handbook to learn how licenses work.

-   **USE Flag Changes**
    When you want to install a package which not only depends on another
    package, but also requires that that package is built with a

    particular USE flag (or set of USE flags) that don't match your
    settings, Portage will warn you:

        The following USE changes are necessary to proceed:
        #required by app-text/happypackage-2.0, required by happypackage 
        (argument) >=app-text/feelings-1.0.0 test

    If `--autounmask` isn't set, then the following may also be shown:

        emerge: there are no ebuilds built with USE flags to satisfy 
        "app-text/feelings[test]".
        !!! One of the following packages is required to complete your 
            request:
        - app-text/feelings-1.0.0 (Change USE: +test)
        (dependency required by "app-text/happypackage-2.0" [ebuild])
        (dependency required by "happypackage" [argument])

    In the given example, the package `app-text/feelings` needs to be
    built with `USE="test"`, but this USE flag is not set on the system.
    To resolve this, either add the requested USE flag to your global
    USE flags or set it for that particular package in the local USE
    flags (more information in Sidenote: USE flags).

-   **Missing Dependencies**

        emerge: there are no ebuilds to satisfy ">=sys-devel/gcc-3.4.2-r4".

        !!! Problem with ebuild sys-devel/gcc-3.4.2-r2
        !!! Possibly a DEPEND/*DEPEND problem. 
               

    Though rarely, the application you are trying to install may depend
    on another package that is not yet available for your system. Unless
    you are mixing branches (categories of the Portage Tree), this
    should not occur and, as such, is a bug that you should report if it

    hasn't been already. There's not much else you can do.

-   **Circular Dependencies**

        !!! Error: circular dependencies: 

        ebuild / net-print/cups-1.1.15-r2 depends on ebuild / 
        app-text/ghostscript-7.05.3-r1
        ebuild / app-text/ghostscript-7.05.3-r1 depends on ebuild / 
        net-print/cups-1.1.15-r2 

    Circular dependencies occur when two or more packages you want to
    install depend on each other and can therefore not be installed.
    This is most likely a bug in the Portage tree and, as such, you
    should resync after a while and try again. Circular dependencies can
    often be avoided by temporarily disabling USE flags that trigger
    optional dependencies.

    You can also check and see if a bug has been filed and, if
    not, report the problem.

*For more error categories and examples, visit:*

http://www.gentoo.org/doc/en/handbook/handbook-x86.xml?part=2&chap=1#doc_chap5

### Sidenote: USE flags

    USE flags are keywords that define the support and dependency info.
    By defining a USE flag, you let portage know to install using
    support for USE flag.

    There are two main types of USE flags:

    1.  *Global flags* are used system wide by several packages. To
        change global USE flags go to `/etc/portage/make.conf`.

    2.  *Local flags* are used by a single package to make
        package-specific decisions. To change local USE flags go to
        `/etc/portage/package.use`.

    There are also temporary USE flags. When installing a program, you
    can set specific USE flags to use just for that installation,
    usually by typing `USE="..."` in front of the `emerge` command.
    However, these temporary USE flags will be lost when you re-emerge
    or update that program.

    The order of priorities for USE flags from lowest to highest is:

    1.  Default USE settings declared in the make.defaults files

    2.  user defined USE flags in make.conf

    3.  user defined USE flags in package.use

    4.  environmental variables (temporary USE flags)


## Resources

"Gentoo Cheat Sheet" http://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet

"A Portage Introduction." Vermeulen, Sven et.al.*Gentoo Linux
Documentation.* gentoo linux. 1 June 2014. Web. Tues 1 July 2014.
http://www.gentoo.org/doc/en/handbook/handbook-x86.xml?part=2&chap=1#doc_chap5



