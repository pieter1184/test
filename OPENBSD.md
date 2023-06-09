Shairport Sync on OpenBSD [AirPlay 2 Not Supported]
----
This is a note about installing Shairport Sync on OpenBSD. Shairport Sync compiles and runs natively on OpenBSD using the `sndio` back end. For general information on `sndio` please follow [this link](http://www.sndio.org).

Unlike FreeBSD, it seems that OpenBSD does not use the directory `/usr/local/etc` as a system configuration directory ("`sysconfdir`") but follows the same practice as Linux in using `/etc` as the default `sysconfdir`.

General
----
This build was done on a default build of `OpenBSD 6.6 GENERIC.MP#5 amd64`. Following [this guide](https://www.openbsd.org/faq/faq15.html), `/etc/installurl` was created with the to refer to the standard repository:
```
https://ftp.openbsd.org/pub/OpenBSD
```
You might have a [closer or faster](https://www.openbsd.org/ftp.html) repository to use instead.

Next, although it may not always be necessary, apply any outstanding system updates and [update the packages](https://unix.stackexchange.com/questions/23579/how-to-apply-updates-on-openbsd-netbsd-and-freebsd).

First, update the system:
```
# syspatch
```
Now update the packages:
```
# pkg_add -Uu
```
Install the Avahi subsystem ([search](https://www.openbsd.org/faq/faq15.html) using, for example, `# pkg_info -Q avahi`). 
```
# pkg_add avahi
```
A number of libraries will be installed to support Avahi, including the D-Bus system.
Enable the D-Bus and Avahi subsystems to [start automatically](http://openbsd-archive.7691.n7.nabble.com/starting-avahi-the-proper-way-td311612.html):
```
# rcctl enable messagebus avahi_daemon 
# rcctl start messagebus avahi_daemon 
```
Building
----

Install the following packages (e.g. using `pkg_add` in superuser mode) needed to download and build Shairport Sync:
```
# pkg_add autoconf automake popt libconfig git
```
Note the versions of `autoconf` and `automake` you choose (`2.69` and `1.16` at the time of writing) and add them as shell variable definitions for -- they could be placed in the user's `.profile` file to be automatically executed at login:
```
export AUTOCONF_VERSION=2.69
export AUTOMAKE_VERSION=1.16
```
Now, download Shairport Sync from GitHub:
```
$ git clone https://github.com/mikebrady/shairport-sync.git
$ cd shairport-sync
```
Next, configure and compile shairport-sync:
```
$ autoreconf -fi
$ ./configure --sysconfdir=/etc --with-avahi --with-ssl=openssl --with-libdaemon \
     --with-sndio --with-os=openbsd
$ make
```
The application is called `shairport-sync`. Check that it's running correctly by executing the following command:
```
$ ./shairport-sync -V
```
This will execute the application and it will return its version information and terminate, for example:
```
3.3.6-libdaemon-OpenSSL-Avahi-sndio-sysconfdir:/etc
```
There is no make install yet -- you're on your own.

Using the `sndio` backend
----
The `sndio` back end does not yet have a hardware volume control facility.
You should set the volume to maximum before use, using, for example, the `mixerctl` command.
