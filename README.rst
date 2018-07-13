==
vm
==

Management tool for KVM virtual machines with ZFS storage

Installation
------------

- ``cd /opt``
- ``git clone https://github.com/makhomed/vm.git``

Upgrade
-------

- ``cd /opt/vm``
- ``git pull``

Usage
-----

Currently ``vm`` understand only two commands, - ``clone`` and ``udev``.

``clone`` usage:

.. code-block:: none

    # /opt/vm/vm clone name:<original-name>:<clone-name> \
        net:<original-bridge>:<clone-bridge> \
        disk:<original-device>:<clone-device>

All command line parameters are required.

``name`` parameter must be defined only once. ``original-name`` is name of existing virtual machine,
``clone-name`` is name of new cloned virtual machine.

``net`` and ``disk`` parameters can be repeated several times,
if virtual machine has multiple virtual network adapters or multiple virtual disk devices.

``vm clone`` designed for special case of virtual machines, which use hardware node bridges
as network devices and use ZFS zvols as block devices.

If you need tool for cloning ordinary virtual machines - you can try to use virt-clone tool.

``udev`` usage:

.. code-block:: none

    # /opt/vm/vm udev

This command iterate all virtual machines and print udev rules
for renaming network interfaces inside router virtual machines.
Router is virtual machine with more than one network interface.

Example
-------

``clone`` example:

.. code-block:: none

    # /opt/vm/vm clone name:template-server:work101-server \
        net:br099:br101 \
        disk:/dev/zvol/tank/kvm-template-server-system:/dev/zvol/tank/kvm-work101-server-system \
        disk:/dev/zvol/tank/kvm-template-server-data:/dev/zvol/tank/kvm-work101-server-data

After executing this command new virtual machine ``work101-server`` will be created.
Original virtual machine ``template-server`` will be used as template.

New virtual machine ``work101-server`` will have new name, new unique uuid,
new unique mac addressed for all virtual network adapters, source bridge name
changed from ``br099`` to ``br101`` and two new virtual disk devices,
``/dev/zvol/tank/kvm-work101-server-system`` and ``/dev/zvol/tank/kvm-work101-server-data``
with all disk content copied from original disk devices.

All other settings will be identical for original and clone virtual machines.

``udev`` example:

.. code-block:: none

    # /opt/vm/vm udev

    # example-router /etc/udev/rules.d/99-br-interfaces.rules

    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="52:54:00:30:81:f5", NAME="br099"
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="52:54:00:85:4f:16", NAME="br100"
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="52:54:00:8f:08:bb", NAME="br101"

These udev rules should be placed in file /etc/udev/rules.d/99-br-interfaces.rules inside virtual machine.

