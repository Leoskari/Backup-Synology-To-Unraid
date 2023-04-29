# Backup Synology to Unraid

Create a file /boot/custom/etc/rsyncd.conf with the following content:

    uid             = root
    gid             = root
    use chroot      = no
    max connections = 4
    pid file        = /var/run/rsyncd.pid
    timeout         = 600

    [backups]
        path = /mnt/user/backups
        comment = Backups
        read only = FALSE

Here above:

-The name "backups" between brackets will be visible as "backup module" from the Synology. You can create several blocks like this one.

-The "path" (here /mnt/user/backups) must exist on your Unraid server (create this one as a shared folder, to be able to access the backup later from any PC)

-Notice: the folder /boot should exist. But you could possibly have to create the subfolders /custom/etc

 

Next, create a file /boot/custom/etc/rc.d/S20-init.rsyncd with the following content:

    #!/bin/bash

    if ! grep ^rsync /etc/inetd.conf > /dev/null ; then
    cat <<-EOF >> /etc/inetd.conf
    rsync   stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/bin/rsync --daemon
    EOF
    read PID < /var/run/inetd.pid
    kill -1 ${PID}
    fi

    cp /boot/custom/etc/rsyncd.conf /etc/rsyncd.conf

Finally, add the following line in the file /boot/config/go :

    #!/bin/bash bash
    /boot/custom/etc/rc.d/S20-init.rsyncd

 

Now, either reboot or execute: 

    bash /boot/custom/etc/rc.d/S20-init.rsyncd
