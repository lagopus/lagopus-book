.. _ref_lagosh:

Using Lagopus CLI (lagosh)
==========================

Starting lagosh
---------------

Lagopus comes with CLI (Command Line Interface) named "lagosh".

* To start lagosh, simply type ``lagosh`` after starting lagopus.

  .. code-block:: console

     $ sudo lagopus
     $ lagosh
     Lagosh>

* You would see error below if Lagopus is not running or did not start properly.

  .. code-block:: console

     Lagosh> show version
     Socket connection refused.  Lagopus is not running?

* Check log output (``-l <logfile>``), configuration files (``lagopus.dsl etc.``) and try running lagopus/lagosh again.

  lagosh has two modes, operational and configuration.

* Prompt ``Lagosh>`` shows you are in operational mode.
* Prompt ``Configure#`` shows you are in configuration mode.

* Enter ``configure`` to move from operational mode to configuration mode.
* Enter ``exit`` or ``quit`` in configuration mode to exit configuration mode and move back to operational mode.
* Enter ``exit`` or ``quit`` in operational mode to exit lagosh and move back to Linux shell.

  .. code-block:: console

     $ lagosh
     Lagosh>
     Lagosh> configure
     Configure#
     Configure# exit
     Lagosh> exit
     $


Using lagosh
---------------

Entering ``?`` or ``help`` will present available commands.

.. code-block:: console

  Lagosh> ?

  Documented commands (type help <topic>):
  ========================================
  configure  help  log  ping  show  ssh  stop  telnet  traceroute

  Undocumented commands:
  ======================
  EOF  exit  pager  quit  shell

Entering ``help <topic>`` or ``? <topic>`` will present brief explanation of the command (topic).

.. code-block:: console

  Lagosh> help show

          Show statistics.
          Usage
                  show bridge
                  show flow
                  show mactable
                  show interface
                  show table

``[tab]`` key is used to auto-complete commands and present available commands and options. For Example:

* Typing ``s`` followed by the ``[tab]`` key will display available commands starting with ``s``.
* Typing ``sh`` followed by the ``[tab]`` key will complete to ``show``.
* Typing ``[tab]`` key twice after ``show[space]`` will present available options for the ``show`` command.

.. code-block:: console

  Lagosh> s[tab]
  show  ssh   stop
  Lagosh> sh[tab]
  Lagosh> show

  Lagosh> show [tab][tab]
  bridge      channel     controller  flow        group       interface   mactable
  meter       port        version

Refer to below for more explanation of each commands.

* `lagosh common commands`_
* `lagosh operational commands`_
* `lagosh configuration commands`_


lagosh common commands
----------------------
This section describes details of commands available in both operational and configuration mode.

* ``help``, ``?``

  * List available commands with "help" or detailed help with "help cmd".
  * See `Using lagosh`_ for more details and examples.

* ``quit``, ``exit``, ``EOF``

  * Entering one of these commands will move you out from current mode.
  * configuration mode -> operational mode -> Linux shell

    .. code-block:: console

       Configure# EOF
       Lagosh> EOF
       $

* ``pager``

  * default: ``off``
  * turn on/off paging with command output has more line than the size of console.
  * Use ``[space]`` to scroll one page.
  * You can also use ``[page up]``, ``[page down]``, ``[up]``, ``[down]`` keys to scroll up / down.

* ``shell``

  * Execute Linux shell command.
  * For exmaple, you can open Lagopus configuration file, run bash and come back to the lagosh CLI like below:

      .. code-block:: console

         Lagosh> shell vi /usr/local/etc/lagopus/lagopus.dsl
         (vi will open "lagopus.dsl")
         Lagosh>
         Lagosh> shell bash
         user@lagopus:~$ ls
         lagopus-0.2.3  v0.2.3.tar.gz
         user@lagopus:~$ exit
         exit
         Lagosh>


lagosh operational commands
---------------------------
This section describes details of each operational commands.

* ``configure``

  * Enter configuration mode.
  * See `lagosh configuration commands`_ for details of configuration mode commands.

  .. code-block:: console

     Lagosh> configure
     Configure#

* ``log``

  * Show log settings.

  .. code-block:: console

     Lagosh> log
     {
         "packetdump": [
             ""
         ],
         "ident": "lagopus",
         "debuglevel": 0
     }

* ``ping <target>``

  * Ping a remote target.
  * Argument <target> is mandatory, and could take IPv4 address or hostname.
  * Enter ``Ctrl+c`` to stop.

  .. code-block:: console

     Lagosh> ping localhost
     PING localhost (127.0.0.1) 56(84) bytes of data.
     64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.018 ms
     64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.020 ms
     ^C
     --- localhost ping statistics ---
     2 packets transmitted, 2 received, 0% packet loss, time 1000ms
     rtt min/avg/max/mdev = 0.018/0.019/0.020/0.001 ms

* ``traceroute <target>``

  * Traceroute to remote target.
  * Argument <target> is mandatory, and could take IPv4/IPv6 address or hostname.

  .. code-block:: console

     Lagosh> traceroute localhost
     traceroute to localhost (127.0.0.1), 30 hops max, 60 byte packets
      1  localhost (127.0.0.1)  0.021 ms  0.006 ms  0.004 ms
     Lagosh>

  .. note::

     ``traceroute`` calles traceroute installed on OS and could return error similar to below if not installed. Exit lagosh and install traceroute in such case.

     .. code-block:: console

        Lagosh> traceroute 127.0.0.1
        sh: 1: traceroute: not found

* ``ssh <target>``, ``telnet <target>``

  * ssh or telnet to a remote target.
  * Attribute <target> is mandatory, and could take IPv4 address or hostname.

  .. code-block:: console

     Lagosh> ssh localhost
     user@localhost's password:
     Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-25-generic x86_64)

      * Documentation:  https://help.ubuntu.com/

      System information disabled due to load higher than 1.0

     Last login: Mon Feb  8 12:44:26 2016 from localhost
     user@lagopus:~$ exit
     logout
     Connection to localhost closed.
     Lagosh>

* ``stop``

  * stop Lagopus.
  * There is no command to start Lagopus. Either exit lagosh or use ``shell`` command to start Lagopus again.

    .. code-block:: console

       Lagosh> shell ps -eo uname,pid,args | grep lagopus
       root      1279 lagopus
       user      1304 sh -c ps -eo uname,pid,args | grep lagopus
       user      1306 grep lagopus
       Lagosh>
       Lagosh> stop
       Lagosh> shell ps -eo uname,pid,args | grep lagopus
       user      1307 sh -c ps -eo uname,pid,args | grep lagopus
       user      1309 grep lagopus


* ``show <arg> [<arg> ...]``

  * Show statistics of objects specified by <arg> in JSON format.
  * Possible arguments:

    * ``bridge``
    * ``controller``
    * ``group``
    * ``mactable``
    * ``port``
    * ``channel``
    * ``flow``
    * ``interface``
    * ``meter``
    * ``version``
  * For some objects, you can specify members by name to limit output.
  * See below for an example using ``show port``.

    .. code-block:: console

       Lagosh> show port
       [
           {
               "name": "port01",
               "state": [
                   "link-down"
               ],
               "is-enabled": true,
               "supported-features": [],
               "config": [],
               "curr-features": []
           },
           {
               "name": "port02",
               "state": [
                   "link-down"
               ],
               "is-enabled": true,
               "supported-features": [],
               "config": [],
               "curr-features": []
           }
       ]

       Lagosh> show port port01
       {
           "supported-features": [],
           "state": [
               "link-down"
           ],
           "config": [],
           "name": "port01",
           "curr-features": []
       }

       Lagosh> show port port01 state
       [
           "link-down"
       ]


lagosh configuration commands
-----------------------------
In lagosh configuration mode, you can view and edit Lagopus software switch configurations.

.. note::

   This feature is still under development.
   Please use this  with special care and check for updates in new version.

* lagosh configuration mode is using **"git"** in background.
* Configure your git email and name before start using the configuration mode.

  .. code-block:: console

     $ git config --global user.email "you@example.com"
     $ git config --global user.name "Your Name"

* To enter configuration mode, enter ``configure`` in operational mode.

  .. code-block:: console

     Lagosh> configure
     Configure#

* Working files used by lagosh configuration mode are located under ``$HOME/.lagopus.conf.d/``. Avoid directly editing files under this directory.
* The directory (``.lagopus.conf.d/``) will be created automatically when you first issue one of the following commands in configuration mode.

  * ``show``
  * ``edit``
  * ``ls``

Typical configuration command lifecycle will be in below order.

#. ``edit`` : edit configuration
#. ``diff`` : check configuration diff from current one
#. ``commit`` : apply to running configuration
#. ``save`` : save to file to be permanent after restart


The remaining part of this section is reference to each configuration commands.

* ``show``

  * Show configuration read from lagopus datastore.

  .. code-block:: console

     Configure# show
     log {
             syslog;
             ident lagopus;
             debuglevel 0;
             packetdump "";
     }
     datastore {
             addr 0.0.0.0;
             port 12345;
             protocol tcp;
             tls false;
     }
     agent {
             channelq-size 1000;
             channelq-max-batches 1000;
     }

     ... snip: more configuration will continue ...

* ``edit [<filename>]``

  * Launch text editor to edit candidate configuration.
  * If ``<filename>`` was not specified, file ``lagopus.conf`` will be opened. It will be created if it did not exist.
  * If you specify ``<filename>``, the file specified will be opened. It will be created if it did not exist.

* ``diff``

  * Show differences from previous configuration.
  * You an find modified mtu is highlighted in ``diff`` command output.

  .. code-block:: console

     Configure# edit
     [master 90c1232]
      1 file changed, 1 insertion(+), 1 deletion(-)
     # modify mtu of interface01 from 1500 to 1000 and exit editor.

     Configure# diff
     diff --git a/lagopus.conf b/lagopus.conf
     index 1a00103..90c5ee2 100644
     --- a/lagopus.conf
     +++ b/lagopus.conf
     @@ -28,7 +28,7 @@ interface {
             interface01 {
                     type ethernet-rawsock;
                     device eth1;
     -               mtu 1500;
     +               mtu 1000;
                     ip-addr 127.0.0.1;
             }
             interface02 {

* ``history``

  * Show differences from previous configuration as git commit history.

  .. code-block:: console

     Configure# history
     commit 90c1232c7ef7407a7851bd71a4784e4e0e79c30b
     Author: Your Name <you@example.com>
     Date:   Mon Feb 8 18:08:36 2016 +0900

     commit 03cc54c1b5e635cb1128e2fe6f2db1948093a0eb
     Author: Your Name <you@example.com>
     Date:   Mon Feb 8 18:08:11 2016 +0900

* ``commit``

  * Apply configuration changes to Lagopus software switch currently running.
  * ``lagopus.conf.dsl`` will be created and running configuration will be stored.

* ``save``

  * Save current configuration to be permanent acress restart.
  * Configuration will be written to the file: ``/usr/local/etc/lagopus/lagopus.dsl``, which is used during Lagopus startup by default.
  * Specifying filename is not supported yet.

* ``ls``

  * List files under ``$HOME/.lagopus.conf.d/``.

* ``load <file>``

  * Load configuration file and commit immediately.
  * Currently raw DSL output is only supported.

* ``set`` (Not implemented yet)

  * Add or modify candidate configuration line.

* ``unset`` (Not implemented yet)

  * Remove candidate configuration line.
