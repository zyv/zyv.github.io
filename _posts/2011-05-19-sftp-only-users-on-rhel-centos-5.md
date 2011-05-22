---
layout: post
title: Setting up SFTP-only users on RHEL/CentOS 5
---

Introduction
------------

Normally, when a user account is created on a Unix system, the user in question can log into the system via `ssh`, forward TCP ports and use the SFTP subsystem if it is enabled in the server configuration. However, at some point, a sysadmin might face the need to create SFTP-only accounts for a number of users. 

A naïve approach would involve creating a new user and setting his or her shell to `/sbin/nologin`. This solution, nevertheless, has two annoying downsides, which might ruin the security of the system, if the users are not trusted enough.

The first one is that the users would still be able to forward ports if this is globally enabled for the trusted user accounts (which includes `sshd` acting as a SOCKS proxy) and the second one is that the users would be able to lurk around the root file system, which certainly cannot be considered as an advantage when the users are not reliable enough.

In order to solve the first problem, one would need to apply certain configuration parameters to a specific subset of user accounts. The second issue can be avoided by instructing the SSH server to `chroot` into the user's directory upon successful login.

One can conveniently apply a bunch of configuration options to specific accounts using the `Match` directive available in the latest versions of the OpenSSH server:

    Match user badguy
        AllowTcpForwarding no
        X11Forwarding no
        ChrootDirectory /srv/sftp
        ForceCommand internal-sftp
    
    Match group sftponly
        ...

However, this directive is only included in OpenSSH 5.x and above, whereas Red Hat Enterprise Linux 5 ships with OpenSSH 4.x. Therefore, an alternative approach will be outlined below.

In what concerns the chrooting, it can be achieved using the `ChrootDirectory` directive. Thankfully, corresponding patches have been backported to OpenSSH 4.x by Red Hat engineers. 

Strictly speaking, it unnecessary to build a proper chroot for SFTP-only users, since OpenSSH includes a built-in SFTP implementation that does not depend upon any external libraries, but if one wants the users to be politely rejected when they try to connect via plain `ssh`, one could just make `/sbin/nologin` work and that is it.

N.B.: Red Hat Enterprise Linux 6 ships a newer OpenSSH version that fully supports the `Match` directive. This means that for RHEL 6 this tutorial would boil down to the configuration snippet for `sshd_config` presented above. Setting up a parallel running SFTP-only `sshd` instance is unnecessary. However, one might still wish to skim through the article for the advice on how to create a proper chroot and read the remarks in the Conclusion.

Implementation
--------------

### Setting up a parallel SFTP-only `sshd` instance

Since `Match` directive is unavailable and the installation of extra unsupported software is to be avoided at all costs, one can bring up a parallel SFTP-only `sshd` instance. The proper way to go would be, of course, to create an extra RPM, i.e. openssh-sftp which installs an additional `init` script and symlinks, but for an one-time deployment this might be an overkill.

First, let us create an `init` script, which is a straightforward modification of the stock `init` script supplied by Red Hat:

    root@box # cat > /etc/rc.d/init.d/sshd-sftponly
    #!/bin/bash
    #
    # Init file for SFTP-only OpenSSH server daemon
    #
    # chkconfig: 2345 55 25
    # description: SFTP-only OpenSSH server daemon
    #
    # processname: sshd-sftponly
    # config: /etc/ssh/ssh_host_key
    # config: /etc/ssh/ssh_host_key.pub
    # config: /etc/ssh/ssh_random_seed
    # config: /etc/ssh/sshd_config-sftponly
    # pidfile: /var/run/sshd-sftponly.pid
    
    # source function library
    . /etc/rc.d/init.d/functions
    
    RETVAL=0
    prog="sshd-sftponly"
    
    # Some functions to make the below more readable
    SSHD=/usr/sbin/sshd-sftponly
    PID_FILE=/var/run/sshd-sftponly.pid
    
    # ZYV
    LOCK_FILE=/var/lock/subsys/sshd-sftponly
    OPTIONS=" -f /etc/ssh/sshd_config-sftponly "
    
    runlevel=$(set -- $(runlevel); eval "echo \$$#" )
    
    start()
    {
        cp -af /etc/localtime /var/empty/sshd/etc
    
        echo -n $"Starting $prog: "
        $SSHD $OPTIONS && success || failure
        RETVAL=$?
        [ "$RETVAL" = 0 ] && touch $LOCK_FILE
        echo
    }
    
    stop()
    {
        echo -n $"Stopping $prog: "
        if [ -n "`pidfileofproc $SSHD`" ] ; then
            killproc $SSHD
        else
            failure $"Stopping $prog"
        fi
        RETVAL=$?
        # if we are in halt or reboot runlevel kill all running sessions
        # so the TCP connections are closed cleanly
        if [ "x$runlevel" = x0 -o "x$runlevel" = x6 ] ; then
            killall $prog 2>/dev/null
        fi
        [ "$RETVAL" = 0 ] && rm -f $LOCK_FILE
        echo
    }
    
    reload()
    {
        echo -n $"Reloading $prog: "
        if [ -n "`pidfileofproc $SSHD`" ] ; then
            killproc $SSHD -HUP
        else
            failure $"Reloading $prog"
        fi
        RETVAL=$?
        echo
    }
    
    case "$1" in
        start)
            start
            ;;
        stop)
            stop
            ;;
        restart)
            stop
            start
            ;;
        reload)
            reload
            ;;
        condrestart)
            if [ -f $LOCK_FILE ] ; then
                stop
                # avoid race
                sleep 3
                start
            fi
            ;;
        status)
            status -p $PID_FILE openssh-daemon
            RETVAL=$?
            ;;
        *)
            echo $"Usage: $0 {start|stop|restart|reload|condrestart|status}"
            RETVAL=1
    esac
    exit $RETVAL
    
    CTRL+D

The configuration file would look as follows (not all directives are necessary, take into account the defaults of the distribution in use):

    root@box # cat > /etc/ssh/sshd_config-sftponly
    # ZYV
    PasswordAuthentication no
    PermitRootLogin no
    PidFile /var/run/sshd-sftponly.pid
    Port 2234
    Protocol 2
    UsePAM no
    
    Subsystem       sftp    internal-sftp
    
    ChrootDirectory /srv/sftp
    AllowTcpForwarding no
    X11Forwarding no
    ForceCommand internal-sftp
    
    CTRL+D

It is important to turn off the use of PAM for authentication in which case `sshd-sftponly` will fall back to reading `/etc/passwd` and `/etc/shadow`, instead of using the pluggable authentication module definitions for authentication sources and methods. Otherwise, one would need to provide an additional PAM configuration file basing upon `/etc/pam.d/sshd` as a template for `sshd-sftponly`, however, it is clearly an overkill if no advanced authentication scheme (e.g. against LDAP) is required.

Then, one needs to create a link to the `sshd` binary, register and start the service:

    root@box # ln -s /usr/sbin/sshd /usr/sbin/sshd-sftponly
    root@box # chkconfig --add sshd-sftponly
    root@box # service sshd-sftponly start

### Creating an SFTP-only user

The next task would be to create a group for SFTP-only users and the users themselves:

    root@box # groupadd sftponly
    root@box # useradd badguy -g sftponly -s /sbin/nologin -m -K UMASK=0022
    root@box # passwd badguy

Here `-g` specifies the main group, `-s` sets the shell, `-m` creates the home directory from a skeleton and `-K` overrides the default options with regards to `umask` (optional).

It is important to set a strong password for the user (which can be immediately discarded) even though the public key authentication is to be used, because otherwise the system would consider this account to be inactive.

Now it is necessary to set up the public key authentication:

    badguy@foo:~$ ssh-keygen -t rsa -b 4096

    root@box # mkdir /home/badguy/.ssh
    root@box # chmod 700 /home/badguy/.ssh
    root@box # cat > /home/badguy/.ssh/authorized_keys
    ...
    CTRL+D
    
    root@box # chmod 600 /home/badguy/.ssh/authorized_keys

### Creating a proper chroot

In order to avoid potential privilege escalation, the chroot and all path components leading up to the chroot have to be owned and only writable by `root`. Additionally, it is necessary to hardlink the shell and some supporting libraries inside the chroot:

    root@box # mkdir -p /srv/sftp/{home,lib,sbin}
    root@box # mkdir /srv/sftp/home/badguy
    
    root@box # chown badguy:sftponly /srv/sftp/home/badguy
    
    root@box # ln /lib/ld-2.5.so /srv/sftp/lib
    root@box # ln /lib/ld-linux.so.2 /srv/sftp/lib
    root@box # ln /lib/libc-2.5.so /srv/sftp/lib
    root@box # ln /lib/libc.so.6 /srv/sftp/lib
    root@box # ln /sbin/nologin /srv/sftp/sbin

Luckily, the OpenSSH server will not allow to use a chroot with wrong permissions.

### Wrapping up

Now it is time to set up a new host entry and try it out:

    badguy@foo:~$ cat >> ~/.ssh/config
    
    Host box
      HostName box.com
      Compression yes
      IdentityFile ~/.ssh/id_rsa
      Port 2234
      User badguy
    
    CTRL+D
    
    badguy@foo:~$ sftp box

Now all these amazing feats would be for nothing if one would not tell the standard `sshd` daemon to deny connections for SFTP-only users and it would happily let them in:

    root@box # cat >> /etc/ssh/sshd_config
    
    # ZYV
    DenyGroups sftponly
    
    CTRL+D
    
    root@box # service sshd restart

Conclusion
----------

There are few additional notes, that I would like to make before closing the article.

First, if you would like to test SFTP access, you need to use `sftp` program as opposed to `scp`. It came as a surprise to me (which probably reflects one of the gaps in my knowledge due to me being an autodidact), but the widely used `scp` program as opposed to the popular beliefs, in fact, *does not* normally implement the SFTP protocol.

It is indeed the case in *some* operating systems, but the canonical version of `scp` implements the [BSD RCP protocol][1] which is tunneled through the Secure Shell (SSH) protocol to provide encryption and authentication. So bear this in mind and use `sftp` instead.

[1]: http://blogs.oracle.com/janp/entry/how_the_scp_protocol_works "How the SCP protocol works"

Another point that is worth being discussed is why one needs a separate root-owned directory tree for a chroot, instead of chrooting directly into users' home directories. This goes back to [CVE-2009-2904][2], when it was discovered that badass users could be able to escalate their privileges via hard links to `setuid` programs that use configuration files within the chroot directory. In the end it was decided that chrooting in user-owned directories actually defeats the purpose of the exercise and additional checks were introduced to restrict the possible sets of permission of the potential chroots.

[2]: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-2904 "MITRE CVE-2009-2904 entry"

Enjoy and if I have missed something in my setup please do let me know!

