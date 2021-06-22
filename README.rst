Open vSwitch v2.15.0 Stochastic Packet Switching
================================================

This fork of Open vSwitch [#ovs]_ implements stochastic switching for packets.
Open vSwitch switches offer an option to split flows travelling through
a switch, which can be used to implement link load balancing.

The problem is that this option is based on a hash of a limited set of
header fields [#fieldlist]_. As most header fields remain constant in any given communication
session, flows will be forwarded to the same link for the entirety of
their session. When simulating load balancing using the hash method, a
very large number of flows are required to ensure proper balancing. This
scales very poorly for simulators.

Stochastic Packet Switching is a quick-and-dirty method to solve this
issue. It implements stochastic switching in the dp_hash method of Open
vSwitch switches [#fieldlist]_.

This version is based off Salahuddin's Stochastic Switching
[#Salahuddin]_ with code rewritten for Open
vSwitch v2.15.0.

Installation
============

Custom Installation
-----------------
Copy ofproto-dpif-xlate.c to the ovs/ofproto directory, then follow the installation
instructions on the Open vSwitch website [#ovs]_. Clone this repository with the following command.

::

    git clone --recurse-submodules git://github.com/tanner-andrulis/stochastic-ovs.git


Quick Installation
------------------
The quick-install script installs on Linux systems with default
options. Quick install is tested and working on the Mininet v2.3.0 [#mininet]_ virtual machine.

To run quick install, issue:

::

    sudo -s
    git clone --recurse-submodules git://github.com/tanner-andrulis/stochastic-ovs.git
    chmod -R 700 stochastic-ovs
    cd stochastic-ovs
    ./quick-install

Running Stochastic Packet Switching
===================================

Open vSwitch daemon can be started as usual.

::

    sudo -s
    export PATH=$PATH:/usr/local/share/openvswitch/scripts
    ovs-ctl stop
    ovs-ctl start

Buckets are set to use stochastic switching by using the default dp_hash selection
method. As this is the default selection method for Open vSwitch routers,
creating any select group will default to stochastic switching [#bugmaybe]_.

To force a dp_hash / stochastic selection method, ensure the following is in your
group declaration.

::

    type=select,selection_method=dp_hash

Testing Stochastic Packet Switching on Mininet
==============================================

This exercise is modified from Salahuddin's exercise [#Salahuddin]_. This exercise involves
building a topology where two hosts can communicate with each other over
two paths. To ensure packets are split properly, paths are set with
weights of 65 and 35, and an iperf session is used to generate a large
number of packets. Finally, packets are counted by Mininet to check that
they are being forwarded in the proper porportions.

Ensure Mininet [#mininet]_ and Python3 [#python]_ are installed.
Then, start the Open vSwitch daemon if it is not started already.

::

    sudo -s
    export PATH=$PATH:/usr/local/share/openvswitch/scripts
    ovs-ctl stop
    ovs-ctl start

Run the Python script located in the examples directory. This script
builds the exercise topology and starts Mininet.

::

    cd example
    ./build_topology.py

Run the Bash script in the examples directory. This script creates the
two flows for packets to traverse. Pay special attention to line 3 of
script.sh. This line invokes stochastic switching.

::

    sh ./add_flows.sh

Open an Xterm window for host h0 and h9.

::

    xterm h0 h9

***In the Xterm window for host h9*** start an iperf server session to
receive and respond to UDP packets from host h0.

::

    iperf -s -u -i 1

***In the Xterm window for host h0*** start an iperf client session to
send UDP packets to host h9 for ten seconds.

::

    iperf -c 10.0.9.2 -u -i 1 -b 50M -t 10

***In the original Mininet window*** check the number of packets sent on
ports 4 and 5 for switch s7. Switch s7 is the separation point for the
two flows; we should see that port 4 sends around 35% of the total
packets, while port 5 sends 65%.

::

    sh ovs-ofctl dump-ports s7 -O OpenFlow13

References
==========
All sites accessed 6/22/2021

.. [#ovs] Website at https://www.openvswitch.org/ and Github at https://github.com/openvswitch/ovs

.. [#fieldlist] Field list section can be found in the ovs-fields section of the Open vSwitch man pages https://www.openvswitch.org/support/dist-docs/

.. [#Salahuddin] https://github.com/saeenali/openvswitch/

.. [#bugmaybe] Additionally, I believe there is a bug in the current Open vSwitch version that makes dp_hash the only available selection method. If this is the case, any and all select groups will use stochastic switching.

.. [#mininet] http://mininet.org/

.. [#python] https://www.python.org/downloads/

License
=======

The following is a summary of the licensing of files in this
distribution. As mentioned, Open vSwitch is licensed under the open
source Apache 2 license. Some files may be marked specifically with a
different license, in which case that license applies to the file in
question.

Files under the datapath directory are licensed under the GNU General
Public License, version 2.

File build-aux/cccl is licensed under the GNU General Public License,
version 2.

The following files are licensed under the 2-clause BSD license.
include/windows/getopt.h lib/getopt\_long.c lib/conntrack-tcp.c

The following files are licensed under the 3-clause BSD-license
include/windows/netinet/icmp6.h include/windows/netinet/ip6.h
lib/strsep.c

Files under the xenserver directory are licensed on a file-by-file
basis. Refer to each file for details.

Files lib/sflow\*.[ch] are licensed under the terms of either the Sun
Industry Standards Source License 1.1, that is available at:
http://host-sflow.sourceforge.net/sissl.html or the InMon sFlow License,
that is available at: http://www.inmon.com/technology/sflowlicense.txt
