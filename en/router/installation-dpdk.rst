.. _ref_installation-dpdk:

How to install DPDK for Lagopus software router
===============================================

This section describes how to install DPDK [#]_ for Lagopus software router.

.. [#] For more information about DPDK, check http://dpdk.org/

Minimum hardware requirement
----------------------------

Lagopus software router requires DPDK which uses multiple CPU cores to achieve high-performance network I/O and processing performance. Therefore, you have to prepare a host machine that meets the following minimum hardware requirements.

* # of CPU cores: **4 or more**
* Memory size: **4 GB or more**
* NIC: DPDK-enabled NICs


Software Versions
-----------------

* OS: `Ubuntu Server 18.04 LTS <http://www.ubuntu.com/download/server>`_
* DPDK: 17.11.1 (LTS)


Installation steps
------------------

Lagopus software router requires to build DPDK in shared lib mode.
Follow below steps to install DPDK in shared lib mode.

* Install Ubuntu Server.

  * Refer to :ref:`ref_virtualbox` for step to install Ubuntu Server on Oracle VM VirtualBox.

* Download and extract DPDK source code

  * http://dpdk.org/download

  .. code-block:: none

   $ wget http://fast.dpdk.org/rel/dpdk-17.11.1.tar.xz
   $ tar xvf dpdk-17.11.1.tar.xz

* Install necessary packages.

  .. code-block:: none

   $ sudo apt update
   $ sudo apt install gcc make python libnuma-dev libelf-dev

* Modify config file to enable CONFIG_RTE_BUILD_SHARED_LIB

  .. code-block:: none

   $ cd dpdk-stable-17.11.1
   ~/dpdk-stable-17.11.1$ cp config/common_base config/common_base.original
   ~/dpdk-stable-17.11.1$ vi config/common_base
   < CONFIG_RTE_BUILD_SHARED_LIB=n
   ---
   > CONFIG_RTE_BUILD_SHARED_LIB=y

* Build and install DPDK

  .. code-block:: none

   ~/dpdk-stable-17.11.1$ make T=x86_64-native-linuxapp-gcc config
   ~/dpdk-stable-17.11.1$ make
   ~/dpdk-stable-17.11.1$ sudo make install


Setup DPDK
----------

To enable DPDK on your environment, you have to enable hugepage.

* Setup Hugepages

Make hugepages available to DPDK. You can setup hugepage in two ways:

1. Manual configuration: Repeate steps after reboot if you select this.
2. Script configuration: Select this to keep it permanent after reboot.

  .. note::

   When configured manually, you need to repeat the same steps after rebooting OS.


1. Manual confguration (undone after reboot)

   Perform the following commands

   .. code-block:: none

      $ sudo sh -c "echo 1024 >  /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages"
      $ sudo mkdir -p /mnt/huge
      $ sudo mount -t hugetlbfs nodev /mnt/huge

2. Script configuration (permanent after reboot)

   * Reserve 1024 pages of 2 MB hugepages in linux by adding the following line in  ``/etc/sysctl.conf``.

     .. code-block:: none

        $ sudo vi /etc/sysctl.conf
        vm.nr_hugepages = 1024

   * Add a directory for hugepages and the following line to ``/etc/fstab`` so that mount point can be made permanent across reboots.

     .. code-block:: none

        $ sudo mkdir -p /mnt/huge
        $ sudo vi /etc/fstab
        nodev /mnt/huge hugetlbfs defaults 0 0

   * Reboot

     .. code-block:: none

        $ sudo reboot

   * Confirm HugePages are configured correctly by the below commands.

     .. code-block:: none

        $ grep -i "HugePages" /proc/meminfo
        AnonHugePages:         0 kB
        ShmemHugePages:        0 kB
        HugePages_Total:    1024
        HugePages_Free:     1024
        HugePages_Rsvd:        0
        HugePages_Surp:        0
        Hugepagesize:       2048 kB
        $ mount | grep huge
        cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
        hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
        nodev on /mnt/huge type hugetlbfs (rw,relatime,pagesize=2M)

NIC (Network Interface Card) assignment
---------------------------------------

The following steps detatch the control and management of NICs from Linux kernel and attach drivers used by DPDK application.

In this example, the host has 4 NICs where we are going to use 3 oth them (``enp0s8``, ``enp0s9``, ``enp0s10``) for Lagopus software router.

  .. note::

   * You need to unbound NIC from kernel (ixgbe driver) before using it with DPDK.
   * You will lose connection to the OS if you unbound NIC used for management plane (ex: ssh).

* Confirm NIC you are planning to use for DPDK are NOT marked ``*Active*``.

  .. code-block:: none

   $ cd ~/dpdk-stable-17.11.1
   ~/dpdk-stable-17.11.1$ ./usertools/dpdk-devbind.py -s
   
   Network devices using DPDK-compatible driver
   ============================================
   <none>
   
   Network devices using kernel driver
   ===================================
   0000:00:03.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s3 drv=e1000 unused= *Active*
   0000:00:08.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s8 drv=e1000 unused=
   0000:00:09.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s9 drv=e1000 unused=
   0000:00:0a.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s10 drv=e1000 unused=

* Load kernel modules
* Disable NICs
* Bind NICs to DPDK driver

  .. code-block:: none

   ~/dpdk-stable-17.11.1$ sudo -s
   ~/dpdk-stable-17.11.1# modprobe uio
   ~/dpdk-stable-17.11.1# insmod build/kmod/igb_uio.ko
   ~/dpdk-stable-17.11.1# ip link set enp0s8 down
   ~/dpdk-stable-17.11.1# ip link set enp0s9 down
   ~/dpdk-stable-17.11.1# ip link set enp0s10 down
   ~/dpdk-stable-17.11.1# ./usertools/dpdk-devbind.py --bind=igb_uio enp0s8 enp0s9 enp0s10

* Confirm (only) NICs for DPDK are listed under "Network devices using DPDK-compatible driver"

  .. code-block:: none

   ~/dpdk-stable-17.11.1# ./usertools/dpdk-devbind.py -s
   
   Network devices using DPDK-compatible driver
   ============================================
   0000:00:08.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000
   0000:00:09.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000
   0000:00:0a.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000
   
   Network devices using kernel driver
   ===================================
   0000:00:03.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s3 drv=e1000 unused=igb_uio *Active*


Next Steps
----------

Refer to :ref:`ref_installation` for steps to install Lagopus software router.

