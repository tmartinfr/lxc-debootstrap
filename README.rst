
lxc-debootstrap
===============

Simple tool to create Debian_ LXC_ containers.

It performs the following operations :

- Creates LVM logical volume and filesystem for the container
- Adds entry to ``/etc/fstab`` and mounts container's filesystem
- Installs Debian using *debootstrap* (with cache to speed up the process)
- Configures networking inside container
- Disables a set of harmful/unused features for a container
- Allows host SSH key to connect to the container's root account
- Creates LXC configuration file
- Starts the container

.. _Debian: http://www.debian.org/
.. _LXC: http://lxc.sourceforge.net/

Requirements
------------

Operating system
^^^^^^^^^^^^^^^^

This tool is compatible with Debian 6.0 and 7.0, as a LXC host or container.

Storage
^^^^^^^

LVM volume group with enough capacity to store the containers.

Networking
^^^^^^^^^^

A linux bridge interface whose IP address serves as network gateway for the
containers.

Example entry in ``/etc/network/interfaces`` : ::

    auto br0
        iface br0 inet static
        bridge_ports none
        address 10.42.0.254
        netmask 255.255.255.0

Packages
^^^^^^^^

These Debian packages must be installed :

- ``lxc``
- ``debootstrap``
- ``lsb-release``

Quickstart
----------

Get the ``lxc-debootstrap`` script.

Create configuration directory : ::

    mkdir /etc/lxc-debootstrap /etc/lxc-debootstrap/containers

Overwrite global variables in ``/etc/lxc-debootstrap/config`` if necessary.
See *Configuration* section below for a full list.

Example (``/etc/lxc-debootstrap/config``) : ::

    DEBIAN_SUITE="wheezy"
    LXC_PATH="/var/lib/lxc"
    BRIDGE_INTERFACE="br0"
    NETMASK="255.255.255.0"
    GATEWAY="10.42.0.254"
    DOMAIN="example.com"

Create container-specific configuration file for an example container *example1*.

Example (``/etc/lxc-debootstrap/containers/example1``) : ::

    IPADDR=10.42.0.29
    HWADDR=42:00:00:00:00:29
    DISKSIZE=5G

As root, execute ``lxc-debootstrap`` : ::

    # ./lxc-debootstrap example1
    
    Container parameters
    --------------------
    lxc_name             example1
    debian_suite         wheezy
    debian_arch          amd64
    lvm_disksize         5G
    lvm_volpath          /dev/mapper/lxc-example1
    net_ipaddr           10.42.0.29
    net_gateway          10.42.0.254
    net_hwaddr           42:00:00:00:00:29
    lxc_rootfs           /var/lib/lxc/example1/rootfs
    lxc_config           /var/lib/lxc/example1/config
    
    Create ? y
    
    debootstrap cache dir already exists
    creating logical volume /dev/mapper/lxc-example1
    creating filesystem on /dev/mapper/lxc-example1
    creating root directory /var/lib/lxc/example1
    adding entry in /etc/fstab
    mount /dev/mapper/lxc-example1 on /var/lib/lxc/example1 using fstab
    populating /var/lib/lxc/example1/rootfs
    networking : setting hostname
    networking : setting DNS resolver
    networking : setting IP configuration
    networking : creating hosts file
    disabling useless tty
    remove pointless services in a container
    disabling root password
    adding SSH keys
    setting APT configuration
    updating packages
    creating configuration file /var/lib/lxc/example1/config
    starting container with lxc-start
    Done.

Now, connect to the container using SSH : ::

    # ssh 10.42.0.29
    The authenticity of host '10.42.0.29 (10.42.0.29)' can't be established.
    RSA key fingerprint is 35:1a:b5:4e:32:c5:0d:4b:34:b1:fe:05:45:b8:30:3a.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.42.0.29' (RSA) to the list of known hosts.
    Linux example1 2.6.32-5-amd64 #1 SMP Sun Sep 23 10:07:46 UTC 2012 x86_64
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    -bash: warning: setlocale: LC_ALL: cannot change locale (en_US.utf8)
    root@example1:~#

Configuration
-------------

Main configuration file is ``/etc/lxc-deboostrap/config``. Per-container
configuration files are ``/etc/lxc-deboostrap/containers/CONTAINER_NAME``.

They overwrite default values (see table below).

Configuration file format is shell script (directly sourced during script
initialization). Variables are defined with KEY=VALUE pairs. Lines can be
commented using # at the beggining of lines.

Each container MUST have these variables defined :

- ``DISKSIZE``
- ``IPADDR``
- ``HWADDR``

Here is a description of all configuration variables :

============= =================================================================
Variable      Description
============= =================================================================
**DISKSIZE**  Container's disk size (e.g. ``5G`` or ``1T``)
**IPADDR**    Container's IP address (e.g. ``10.42.0.1``)
**HWADDR**    Container's MAC address (e.g. ``42:00:00:00:00:29``)
LXC_VG_NAME   LVM volume group where container's logical volume are allocated
LXC_PATH      LXC root path where container's mount point are created
DEBIAN_MIRROR Debian mirror URL
DEBIAN_SUITE  Debian distribution codename
DEBIAN_ARCH   Debian architecture
DOMAIN        Domain name, used in /etc/hosts and /etc/resolv.conf if defined
NETMASK       Network mask of container's network interface
GATEWAY       Network gateway of container's network interface
BRIDGE_IF     Network bridge interface name on host system
DNS_RESOLVER  Name server IP address
SSH_PUBFILE   SSH public key to allow to connect to the container root account
CACHE_ROOT    Directory where debootstrap cache is stored
FSTYPE        Container's filesystem type
============= =================================================================

And their default values :

============= =================================================================
Variable      Default value
============= =================================================================
**DISKSIZE**  Mandatory, no default value
**IPADDR**    Mandatory, no default value
**HWADDR**    Mandatory, no default value
LXC_VG_NAME   ``lxc``
LXC_PATH      ``/var/lib/lxc``
DEBIAN_MIRROR ``http://ftp.debian.org/debian/``
DEBIAN_SUITE  Same as host (e.g. ``squeeze`` or ``wheezy``)
DEBIAN_ARCH   Same as host (e.g. ``amd64`` or ``i386``)
DOMAIN        None
NETMASK       ``255.255.255.0``
GATEWAY       ``10.42.0.254``
BRIDGE_IF     ``br0``
DNS_RESOLVER  ``10.42.0.254``
SSH_PUBFILE   ``/root/.ssh/id_rsa.pub``
CACHE_ROOT    ``/var/cache/lxc-debootstrap``
FSTYPE        ``ext4``
============= =================================================================

FAQ
---

Q : Can I install additional packages using deboostrap ``--include`` option ?

A : No, it could break the debootstrap cache

TODO
----

- Check executed as root
- Check lxc_name is short
- Check directories do not exist
- Check DISKSIZE, IPADDR, HWADDR are defined, and only container-defined
- Option to force yes
- Option to build all containers
- Option to rebuild/update the debootstrap cache
- Option to remove container
- Source post-hook scripts in /etc/lxc-debootstrap/post.d/
- Auto-detect values of NETMASK, GATEWAY, DNS using bridge configuration

Author
------

Copyright 2013 Thomas Martin thomas@oopss.org

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see http://www.gnu.org/licenses/.

