openwrt multi vpn
=================

Conditional multiple vpns for openwrt using mwan3.

This role sets up multiple vpn clients on the same openwrt host, together with rules
specifying which hosts should use which vpn. It only writes configuration files,
and does not actually start services, since I did not bother to find the right service restart
order. So run the role, then either restart services or reboot.

**Beware**: It was only tested on one router. It can probably destroy your config, and messing
with your live internet gateway is always scary. Be sure to backup your configuration before you start,
I don't guarantee anything about this role. I also only tried it with https://privateinternetaccess.com/ .

This role is heavily based on an excellent blog post by Leow Kah Man, and you should
definitely read it before trying.

https://www.leowkahman.com/2016/06/19/conditional-multiple-openvpn-routing-hostname-ip/

It also builds upon the nice openwrt role by gekmihesg.

Requirements
------------

 * The openwrt host **must** be in a group called openwrt, since that is how gekmihesg/openwrt works.
 * Download the `ca.rsa.2048.crt` `crl.rsa.2048.pem` files from the provider to `/etc/openvpn`.
 * Create a file called `/etc/openvpn/pass` where the first line is the user and the second is the password.

Role Variables
--------------

`vpns` and `mac_denies`. See example playbook below.

Dependencies
------------

 * gekmihesg/openwrt. Tested with commit 4eda6e5, which was master when this was published.

Example Inventory
-----------------

    [openwrt]
    192.168.0.1

    [openwrt:vars]
    ansible_remote_tmp=/tmp/ansible

Example Playbook
----------------

    - hosts: openwrt
       gather_facts: false
       tasks:
       - include_role:
           name: openwrt-multi-vpn
         vars:
           vpns:
             - name: nycvpn
               # You get the remote host from your vpn provider.
               remote: us-newyorkcity.privateinternetaccess.com 1198
               # Each vpn must have a different metric value (20, 21, 22..).
               metric: 20
               # Destination domains to tag with ipset.
               destdomains:
                 - example.com
               # Use this vpn for any traffic from this ip address.
               srcip: 192.168.0.60/32
           # These macs are denied direct access to wan, and must use a vpn,
           # usually correlates to srcip.
           mac_denies:
             - 00:11:22:33:44:55


License
-------

Apache V2.0

Author Information
------------------

https://github.com/jarondl/openwrt-multi-vpn
