openvswitch (ansible role)
=========

role that installs and configures openvswitch

Requirements
------------

systemd with active networkd deamon.

Role Variables
--------------

Optional values can be omitted completely. Links provided show original documentation for systemd configuration files.

     openvswitch_bridge_name: name for the openvswitch bridge (interface)
     openvswitch_access_port_name: name for the interface over which the machine will communicate at layer 3
     openvswitch_access_port_vlan: vlan to assign to the access port.
     openvswitch_physical_port_name: array of physical interfaces to add to the openvswitch bridge

All relevant values are documented at: https://www.freedesktop.org/software/systemd/man/systemd.network.html

     openvswitch_systemd_networks: array of hashes that configure systemd networkd settings to start openvswitch interfaces
       (required) .priority: sets the initialization order of interfaces as parsed by networkd.
       (required) .match_name: the name of the open vswitch interface to assing an ip to.
       (optional) .network_dhcp : decide how to start dhcp. (possible options: yes,no,ipv4,ipv6)
       (optional) .network_address: ip address in cidr notation
       (optional) .network_gateway: ip of gateway
       (optional) .network_domains: list of domain suffixes to search
       (optional) .network_dns: array of ips of dns servers
       (optional) .link_mtubytes: manually set link mtu
       (optional) .dhcp_usedns: set to true if want to use dhcp assined dns address (they will append to any manually specified ones)

All relevant values are documented at: https://www.freedesktop.org/software/systemd/man/systemd.link.html

     openvswitch_systemd_links:
       (required) .priority: sets the initialization order of interfaces as parsed by networkd.
       (required) .match_originalname: name of physical link to ensure is brought up before open vswitch tries to use it
       (optional) .macaddresspolicy: use persistent to make sure the mac address doesn't change between reboots. defaults to persistent.
       (optional) .namepolicy: override interface naming policy to prevent arch linux from renaming links. defaults to 'kernel database onboard slot path'
       (optional) .macaddress: manually set a mac address
       (optional) .mtubytes: manually set the link mtu
       (optional) .name: 'The interface name to use in case all the policies specified in NamePolicy= fail, or in case NamePolicy= is missing or disabled.' per the link above.
       (optional) .bitspersecond: manually set link speed
       (optional) .duplex: manually set the duplex
       (optional) .wakeonlan: set wake on lan policy

Dependencies
------------

systemd.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role:openvswitch,
             openvswitch_bridge_name: ovs-bridge,
             openvswitch_access_port_name: ovs-access-int,
             openvswitch_access_port_vlan: '2',
             openvswitch_physical_port_name: ['eno1','eno2'],
             openvswitch_systemd_networks: [ { 'priority': 20,
                                               'match_name': ovs-access-int,
                                               'network_dhcp': off,
                                               'network_address': 10.0.9.55/24,
                                               'network_gateway': 10.0.9.1,
                                               'network_domains': ['domain.lan',],
                                               'network_dns': ['8.8.8.8','8.8.4.4'],
                                               'dhcp_usedns': 'true'
                                             },
                                           ],
             openvswitch_systemd_links: [ { 'priority': 10,
                                            'match_originalname': 'eno1',
                                            'macaddresspolicy': 'persistent',
                                            'namepolicy': 'database kernel onboard slot path',
                                            'wakeonlan': 'magic',
                                          },
                                          { 'priority': 10,
                                            'match_originalname': 'eno2',
                                            'macaddresspolicy': 'persistent',
                                            'wakeonlan': 'magic',
                                          },
           }

License
-------

GPLv3

Author Information
------------------

Yevgeniy Kuksenko
