.. _ref_introduction:

Introduction to Lagopus software switch
=======================================================

Lagopus software switch is a yet another OpenFlow 1.3 software switch implementation leveraging DPDK for high-performance packet processing to realize software-defined networking.

Lagopus software switch is designed to archive over-10-Gbps packet processing throughput and forwarding performance on standard PC server for flexible and complex traffic flow handling. Lagopus software switch is supposed to be controlled by one or multiple SDN controller with OpenFlow switch 1.3 protocol. In addition, Lagopus software switch can interwork with the existing routing stack, such as Quagga, in other words, hybrid SDN is going to be realized. Of curse, Lagopus software switch can run in stand-alone mode with Layer-2, and Layer-3 configuration.

Lagopus software switch provides flexible and programmable dataplane for commonly-used network packet frame format. Many packet frame formats are handled in Lagopus software dataplane: VLAN, QinQ, MAC-in-MAC, MPLS, Provider-Backbone-Bridge (PBB), IPv4, IPv6, ARP, ICMP, UDP and TCP are supported. Lagopus software dataplane also provide general tunnel encapsulation and decapsulation for overlay-networking for data center and wide-area network.


Hardware requirement
------------------------

Lagopus software switch can run on Intel PC servers with NIC as well as Intel Desktop PC.

* CPU: Intel Xeon, Core, Atom
* NIC: `DPDK Supported NIC <http://dpdk.org/doc/nics>`_  [#]_
* Memory: >= 2GB

.. [#] : DPDK supported NIC is only required when using DPDK and not when running in raw-socket mode.

Lagopus can also run on a virtual machine on `Oracl VM VirtualBox <https://www.virtualbox.org/>`_, `KVM <http://www.linux-kvm.org/>`_, `VMware Workstation, Fusion and ESXi <http://www.vmware.com>`_, Microsoft Hyper-V.
Instruction of Lagopus setup on a virtual machine is described :ref:`ref_virtualbox` and :ref:`ref_installation-kvm`.


Software requirement
------------------------

Lagopus software switch can run on many Linux distribution, FreeBSD and NetBSD [#]_

* Linux

  * Ubuntu
  * CentOS
  * RedHat

* FreeBSD
* NetBSD

.. [#] : Most features are implemented for the following operating systems, but some features are not implemented due to lack of OS support.

Support
-------

- `Lagopus software switch official site: https://lagopus.github.io/ <https://lagopus.github.io/>`_
- Lagopus mailing list: lagopus-devel@lists.sourceforge.net and `archive <https://sourceforge.net/p/lagopus/mailman/lagopus-devel/>`_


Development
-----------

Your contribution is very welcomed. Please submit your patch code using `github "Pull Requests" <https://help.github.com/articles/using-pull-requests/>`_. If you find any bug, let us know by `github Issues page <https://github.com/lagopus/lagopus/issues>`_.

For further information about how to get assistance, submit issues and pull request, refer to Lagopus GitHub Wiki Page located here : https://github.com/lagopus/lagopus/wiki

License
---------

The code of Lagopus software switch is freely available under the Apache version 2.0
