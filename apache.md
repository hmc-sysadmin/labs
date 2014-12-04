# Web Servers

*Originally created by Huting Lin and Jeff Milling*

[Name resolution]: http://technet.microsoft.com/en-us/library/cc958812.aspx
[Elinks]: http://elinks.or.cz/documentation/html/manual.html-chunked/ch03.html

This week, you'll configure and secure a web server on your VM that is capable
serving multiple web sites. In all, you'll serve pages from three sites:

   - `vm(num).sys.cs.hmc.edu`: the "default" website for your machine
   - `www.(your server).net`: a virtual host that your machine can serve
   - `admin.(your server).net`: a password-protected virtual host

You'll also configure one of the sites to automatically serve pages via an
encrypted channel, over SSL.

## Background information

Here is some reading and background information that may be useful for this lab:

Name resolution
  : [A nice article][Name resolution] and reminder about how name resolution works, plus
    instructions for configuring name resolution on UNIX and Windows.

Apache
  : Apache HTTP server has been the most prevalent web server software on the
    internet since 1996. Made by The Apache Software Foundation, Apache HTTP server
    is an open source HTTP web server with support for a wide variety of web
    frameworks and services. Learning Apache and Apache configuration is an
    essential part of any system administrator's training.

elinks
  : A [console-based web browser][Elinks] that you can use on your VM to visit web pages.

## Getting started

  1. Write down the URLs for the three websites you'll create. Fill in the
  blanks with the appropriate information:

     - `vm(num).sys.cs.hmc.edu`
     - `www.(your server).net`
     - `admin.(your server).net`
 
  1. On your VM, open elinks and visit each site. You should be able to confirm
  that each site is unreachable.

## Configure and start the web server on your VM

### Configure your system

1. Portage, Gentoo's package management system, uses USE flags to help
    determine which dependencies to install in order to support your
    packages. View\
    `/etc/portage/make.conf` to check that support is enabled for the
    apache2 package on your machine. apache2 should be included in the
    USE variable:

        USE="... apache2 ..."

    If it's not, edit the file to add the apache2 USE flag, then update the
    system by running the following command:

        sudo emerge --ask --changed-use --deep @world
        
2. Apache uses localhost to connect internally, because of this, you 
     need to make sure your /etc/hosts file is correctly configured.
     Open `/etc/hosts` and edit the line:

        127.0.0.1       SysAdmin_VM[#] localhost ...
        
     Replace the `#` with your VM's number

### Configure apache
Apache has two main configuration files in the system:

  -   Gentoo's apache2 init script configuration file
        `/etc/conf.d/apache2`

  -   Apache server's conventional configuration file
        `/etc/apache2/httpd.conf`

Take a look at `/etc/conf.d/apache2`. The only active line in this file is:

    APACHE2_OPTS="-D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D LANGUAGE" 

*(the one on your system may be a bit different)*

This line defines options that will be interpreted by the
various configuration files using the `<IfDefine option-name>`
statement to activate or deactivate some part of the whole
configuration, usually modules. Modules are first and
third-party patches to Apache that install more functions for
Apache to use.

  1. Modify `/etc/apache2/httpd.conf` to add a `ServerName` configuration
        ```      
        ServerName vm(num).sys.cs.hmc.edu
        ```
     This line can go anywhere in the file.

Apache server's conventional configuration file, `httpd.conf`, 
is actually only an entry file, or an entry point for
users in a file form, since Apache's whole configuration is
split in many files in the `/etc/apache2/` directory, assembled
and connected together using the **Include** directive.

NOTE: Module configuration files (files in
`/etc/apache2/modules.d`) almost always start with
`<IfDefine module-name>`. For this reason, the content of those
files will ONLY be assembled with the rest of the configuration
if the `-D module-name` flag in the `APACHE2_OPTS` variable
mentioned above is set to the matching option. The
`00_default_settings.conf` configuration file is an exception to
this rule since it doesn't start with an `IfDefine` statement
and therefore is always included in the resulting configuration.

### Start Apache

1. Run the following command to start up your web server

  ```
  sudo rc-service apache2 start
  ```

## Visit your website
Try to visit the URL for `vm(num).sys.cs.hmc.edu`, both on your VM (using elinks)
and in a browser on your own machine. 

*Note:* The websites on the VMs are accessible only to visitors with a Claremont
IP address.

If you're not able to reach the website
from your VM, try editing `/etc/hosts` so that it looks something like this:

    ```
    127.0.0.1       localhost
    ::1             localhost
    (your IP)       vm(num).sys.cs.hmc.edu
    ```

If you're not able to reach the website from your own computer, you can try
[configuring the local name database][Name resolution] in a similar way.

## Creating Name-based Virtual Hosts

"Name-based virtual hosts" is a fancy name for what we implement to
allow our web servers to serve multiple websites. Without virtual hosts,
your webserver would only respond to http://localhost and/or
(your-vm).sys.cs.hmc.edu. We create name-based virtual hosts so that our server
responds to other prefixes, like www.(your-server).net, or
admin.(your-server).net. For (your-server), you can name it anything you
want. *For more information visit:*
[http://gentoovps.net/apache-vhost](http://gentoovps.net/apache-vhost/).

Here's how to set up a virtual host on your machine:

1.  Go to the directory that stores all the configuration files of
    virtual hosts.

        cd /etc/apache2/vhosts.d

1.  Create the file `net.(your server).www.conf` and, in the file, type
    the following. Try to think about the purpose of each line as you
    write them. Each line is explained below.

        <IfDefine DEFAULT_VHOST>
         <VirtualHost *:80>
          ServerName www.(your server).net
          ServerAlias (your server).net
          ServerAdmin (name)@(your server).net
          DocumentRoot "/var/www/net.(your server).www/htdocs"
          <Directory "/var/www/net.(your server).www/htdocs">
           Options Indexes FollowSymLinks
           AllowOverride All
           Order allow,deny
           Allow from all
          </Directory>
         </VirtualHost>
        </IfDefine>

    - The `<IfDefine test>...</IfDefine>` section is used to mark
    directives that are conditional. The directives within an
    `<IfDefine>` section are only processed if the test is true. If
    test is false, everything between the start and end markers is
    ignored.

    - The `<VirtualHost>...</VirtualHost>` section encloses a group of
    directives that apply only to a particular virtual host. It also
    states which IP address(es) the vhost replies to.

        -   `ServerName` is pretty self-explanatory in that it
            establishes the name of the server. `ServerAlias` is also
            self-explanatory in that it sets the other possible url's
            that can redirect to this particular virtual host.

        -   `ServerAdmin` sets the contact address that the server
            includes in any error messages it returns to the client.

        -   `DocumentRoot` sets the directory from which httpd will
            serve files.

    - The `<Directory>...</Directory>` section encloses directives
    that will apply to the named directory, sub-directories of that
    directory, and the files within the respective directories.

        -   The `Options` directive controls which server features,
            usually those considered extra features, are available in a
            particular directory.

        -   `AllowOverride` establishes which directives in `.htaccess`
            files are allowed to override earlier configurations.

        -   The `Order` directive controls the default access state of
            the site/page, and the order in which the `Allow` and `Deny`
            directives are evaluated.

        -   `Allow` controls which hosts can access an area of the
            server, as its name suggests.

    *For more detailed explanations and examples, visit:*\
    <http://httpd.apache.org/docs/2.2/mod/core.html#ifdefine>\
    *and*\
    <http://httpd.apache.org/docs/2.2/mod/mod_authz_host.html#allow>

1.  Make a directory/site root for the new server:

        mkdir -p /var/www/net.(your-server).www/htdocs

    NOTE: The `-p` option tells mkdir to create the parent directories
    of the target directory if they don't exist.

1.  In that directory, create an easy `index.html` and add the following to
    the file:

        <html>
         <head> 
          <title>www</title>
         </head>
         <body>
         <h1>This is the www page</h1>
         </body>
        </html>

1.  To locally associate the IP address with the server name, edit
    `/etc/hosts` and add the following values for 127.0.0.1.

        127.0.0.1 ...www.(your server).net

1.  Reload apache's configuration with the following command:
    ```
    sudo rc-service apache2 reload
    ```
    **Every time you reconfigure apache, you'll need to reload the configuration
    .**

1.  Check to see if everything's working by using a browser to
    go to `www.(your server).net`. **You'll need to use a _local_ browser 
    (i.e., a browser on your VM) to visit the site because only your VM can 
    resolve the URL.**

Let's also create an administrator site that corresponds to the site you just
created . Call it `admin.(your server).net`. **Follow the steps above to create
a virtual host for this site.** Remember to edit the new admin config file,
specifically the **ServerName, DocumentRoot, *and* Directory** directives to
reflect the change from a www page to an admin page. **ServerAlias** can be
deleted. Also, don't forget to make the necessary changes to the HTML file as
well. When you're done, visit the site in a browser to make sure it's up.

## Activity with a parter
Grab a partner. From your machine, make sure you can visit:

   - your partner's VM website (`http://vm(number).sys.cs.hmc.edu`)
   - your partner's virtual host (`http://www.(their server).net`)
   - your partner's virtual admin host (`http://admin.(their server).net`)

If you're unable to visit any of these sites, edit `/etc/hosts` and add a line
like the following:

    partner-ip vm(number).sys.cs.hmc.edu www.(their server).net admin.(their server).net

## Apache Security Configurations

*Note: you'll need your partner for part of this work*

As a system administrator it is your job to not only make sure that the
system is fully functioning, but also to make sure the system is secure
and safe. There are a wide variety of nefarious exploits and incorrect
usages that open up when your system is hosting a web service, including
Apache. But if properly configured, it is possible to prevent a majority
of the security vulnerabilities in existence today.

### Step 1: Restrict access based on IP address

We'll restrict access to `admin.(your server).net` so that only certain IP
addresses/machines can access the site. Since our website was just created, it
has some of the weakest security measures available right now. As such, we'll
just start with something simple and build up from there.

  1. Edit the configuration file for the admin virtual host. In the
  `<Directory...>` section, change the `Allow` directive from **all** to the  IP
  addresses that you want to allow access. In this case, change it to
  **127.0.0.1**. When you're done, reload apache. Now apache will only
  serve the site's files to the localhost address.

  1.  Make sure that your partner can no longer access `admin.(your server).net`.

  1. Add your partner's IP's to allow them to access your site. Make sure they
  can do it. Now apache will only serve the site's files to the localhost 
  address and to your partner's IP address.


Because of how simple it is to change one's own IP address, this form of
security is almost useless unless combined with with password protection and
encrypted traffic, to make sure no one else can overhear the exchange of
information (and passwords!).

### Step 2: Protect your site with a password

  1. In `/var/www/net.(your server).admin/htdocs`, create a file called
  `.htaccess`. This file is located here instead of the traditional directory
  for config files because, as a rule of `.htaccess` files, they must be located
  in the directory they would be affecting. So it will go in the `htdocs`
  directory, where the HTML files will keep it company.

  2.  In `.htaccess`, type the following (each line is explained
    below):

        ```
        AuthUserFile /var/www/net.(your server).admin/conf/admin-auth
        AuthGroupFile /dev/null
        AuthName "Admin only area"
        AuthType Basic

        <Limit GET POST>
         require valid-user
        </Limit>
        ```

        - `AuthUserFile` tells the server which text file contains the
            list of users and passwords for authentication.
            `AuthGroupFile` has a similar function but for groups
            instead.

        - The `AuthName` directive sets the name of the authorization
            realm for a directory, giving the client a hint as to which
            user-password combinations to use.

        - Like its name suggests, `AuthType` sets the type of
            configuration the server would have.

        - `<Limit>` and `</Limit>` restrict enclosed access controls
            to only certain HTTP methods. *Require* selects which
            authenticated users can access a resource.

        *For more information and examples, visit:*\
        <http://httpd.apache.org/docs/2.2/mod/mod_authn_file.html>\
        *and*\
        <http://httpd.apache.org/docs/2.2/mod/core.html#require>

  3. Security of files is important too. So, as a precaution, change the
  permissions and ownership of .htaccess so you and apache can access the file,
  but only the owner can write in it. The backslash indicates that the line
  extends to the next line.

            chgrp apache /var/www/net.(your server).admin/htdocs/.htaccess

            chmod 640 /var/www/net.(your server).admin/htdocs/.htaccess

  4. Make the directory `conf` in the admin page's directory in `/var/www/net
  .(your-server).admin/`. Once that's done, we will create the password file
  that will contain all the user-password combinations.
        ```
        htpasswd -c /var/www/net.(your server).admin/conf/admin-auth (USER)
        ```

  Later, if you'd like to add more users, you would use the same command but
  without the `-c` option.
        ```
        htpasswd /var/www/net.(your server).admin/conf/admin-auth (USER)
        ```

  5.  Now, protect the password file the same way as you did for
        `.htaccess`

  6. Now try it out! You should be greeted with a login page instead this time
  around, meaning that the configuration changes worked. P.S. You may have to
  reload the page (ctrl-R) in order for the page to reflect the fact that your
  user-password combination was correct.

### Step 3: Provide an encrypted connection to the site.

Though your page is now somewhat secure, the connection between a machine and
the server is still unguarded. A secure, encrypted ssl connection is called on
by typing `https://` in the url. However, if you try it now, you would only get
the localhost default page since the admin page is not configured to listen to
port 443. Now, since OpenSSL should already be installed, we can go straight
to generating the encryption key and certificate:

  1. In your home directory, generate an SSL key

              openssl genrsa 2048 > net.(your server).key

  1. In your home directory, generate an SSL certificate
        ```
        openssl req -new -x509 -nodes -sha1 -key net.(your server).key > net.(your server).crt
        ```
  
  In the above command, make sure that the option -sha1 contains the number one
  and not a lowercase l. When it asks for information, you can fill in however
  much you want.

  1. Move the key and certificate files to `/etc/apache2`

  1. Check to make sure `APACHE2_OPTS` in `/etc/conf.d/apache2` has
        `SSL` and `SSL_DEFAULT_VHOST`.

  1. `NameVirtualHost` is a required directive if one wants to configure name-
  based virtual hosts. It specifies the IP address on which the server will
  receive and answer requests for the virtual hosts, usually used when more than
  one vhost listens to the same IP address. So, in
  `/etc/apache2/vhosts.d/00_default_ssl_vhost.conf`, under the **Listen** line
  in the file, add the following:
        ```
        NameVirtualHost *:443
        ```

  1.  Next, we need to configure the admin page to actually have a
        virtual host to answer an SSL connection request. This is
        because the virtual host that is already established only listens to
        requests on port 80, while SSL connections use port 443. Also,
        with its different configurations, the "non-secure" vhost and
        the SSL vhost have to remain separate. So, enter the config file
        for the admin page and, directly after the line
        `</VirtualHost>`, add the following:

            <IfDefine SSL>
             <IfModule ssl_module>
              <VirtualHost *:443>
               SSLEngine on
               SSLCertificateFile /etc/apache2/net.(your server).crt
               SSLCertificateKeyFile /etc/apache2/net.(your server).key
               
               ServerName admin.(your server).net
               
               SSLOptions StrictRequire
               SSLProtocol all -SSLv2
               
               SSLCipherSuite RC4-SHA:AES128-SHA:HIGH:!aNULL:!MD5
               SSLHonorCipherOrder on
               
               DocumentRoot "/var/www/net.(your server).admin/htdocs"
               <Directory "/var/www/net.(your server).admin/htdocs">
                SSLRequireSSL
                Options Indexes FollowSymLinks
                AllowOverride All
                Order allow,deny
                Allow from all
               </Directory>
              </VirtualHost>
             </IfModule>
            </IfDefine>

        - The `<IfDefine>` is mentioned before, and the `<IfModule>`
            section encloses directives that are conditional on the
            presence of a specific module.

        - `VirtualHost` still has the same function, though in this
            case it considers the SSL default port of 443 instead.

            -   The `SSLEngine` directive toggles the usage of the
                SSL/TLS Protocol Engine, which is responsible for
                creating SSL/TLS connections.

            -   For `SSLCertificateFile` and `SSLCertificateKeyFile`,
                their names are self-explanatory. They establish the
                full path to the SSL key and certificate files created
                earlier.

            -   `SSLOptions` is used to control various run-time options
                on a per-directory basis

            -   Just like its name, `SSLProtocol` is used to control
                which SSL protocol mod\_ssl would use when establishing
                the server environment.

            -   `SSLCipherSuite` establishes which Cipher Suites are
                available for negotiation in a SSL handshake. Meanwhile,
                `SSLHonorCipherOrder`, when enabled, means that
                connections will prefer the server's cipher preference
                order when choosing a cipher.

        - In the `<Directory>` section, whose file condition should
            stay the same, the `SSLRequireSSL` directory denies the
            client access when SSL is not used for the HTTP request
            a.k.a. HTTPS is not used, though it isn't as effective as
            the next configuration.

            **Warning**: Notice that you allow connections over SSL from *all*
            IP addresses. You can change `Allow from all` to allow from only
            certain IPs, as you did for the non-SSL version.

        NOTE: There should be two lines of `</IfDefine>` at the end of
        the config file. This is because the new `<IfDefine>` section is
        nested within our `<IfDefine DEFAULT_VHOST>` section.\
        *For more information and examples, visit:*\
        <http://httpd.apache.org/docs/2.2/mod/mod_ssl.html>

Now you can try this out by using `https://` in place of `http://`. Since the
certificate created is extremely sketchy, some browsers might ask if you really
want to continue. Follow the steps to progress, and you should find yourself at
the admin page once again. With the current settings, you will only go through a
ssl-secured network when `https://` is used. `http://` will still go through
normal, unsecure connections.

If you encounter the SSL error "unable to get local
issuer certificate," there is a way to fix it! Go to the file
`00_default_ssl_vhost.conf` and change the
`SSLCertificateKeyFile` and `SSLCertificateFile` to the path of
certificate and key you created.

Now you can set up your site so that **all attempts to access the site will go
through SSL.**

  1.  In the `<VirtualHost *:80>`
        section of the admin's conf file, add the following lines:

            RewriteEngine On
            RewriteCond %{HTTPS} !on
            RewriteRule ^/(.*) https://%{SERVER_NAME}%{REQUEST_URI} [R]

        - Like SSLEngine, the `RewriteEngine` directive enables or
            disables the runtime rewriting engine.

        - `RewriteCond` defines the condition under which rewriting
            will take place, while `RewriteRule` defines the rules for
            the rewriting engine.

        *For more information and examples, visit:*\
        <http://httpd.apache.org/docs/current/mod/mod_rewrite.html>
  
  1.  Now, when you try to access `http://...`, it should
        automatically connect to `https://...`. In other words, you will
        always go through a secure connection!

Great! Now you have configured a (relatively) secure website, with an
even more secure admin site. 
