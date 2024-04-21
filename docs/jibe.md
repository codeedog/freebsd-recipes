# `jibe` jail network shortcut tool

`jibe` provides simplified jail vnet network configuration. It covers
common jail network configurations for DHCP, static IP, Bridge, and
epair. It's intended to be executed within the jail configuration file
context during prestart, start and poststop sections.

## Examples

### DHCP

Assume you have a host or parent jail interface connected to a
(virtual) network with a DHCP server. This formulation adds that
interface to a bridge, creates an epair also added to that bridge,
passes the other end of the epair to the jail and initiates the DHCP
request for the jail's epair endpoint. Note: it's possible to create
the bridge and add the host interface one time in `/etc/rc.conf`
rather than at every jail startup.

````
/etc/jail.conf.d/dhcp.conf:

  $e0 = "e0b_${name},dhcp,b/pub_bridge";

  # Network
  vnet.interface=e0b_${name};

  exec.prestart+="jibe pre  $e0";
  exec.start   +="jibe bind $e0";
  exec.poststop+="jibe del  $e0";
  exec.prestart+="ifconfig pub_bridge addm eth0 up";
````

`jibe` generates a MAC hardware address to allow consistent address
assignment across requests. This address is generated from the jail
name and may cause collisions. Caller may specify MAC address through
the configuration as follows:

````
  $e0 = "e0b_${name},dhcp,b/pub_bridge,02:92:71:65:e1:0b";
````

Note: the last hex section is ignored as epair's require this to be
"0a" and "0b" matching the epair endpoints.


### Assigned IPv4 w/ bridge

This formulation assigns a static IP to one end of the epair and
places the other into a private bridge, presumably for virtual switch
communications between jails. It also sets the default gateway.

````
/etc/jail.conf.d/inet.conf:

  $e2 = "e2b_${name},172.31.40.17/24,b/priv_bridge,d=172.31.40.1";

  vnet.interface=e2b_${name};

  exec.prestart+="jibe pre  $e2";
  exec.start   +="jibe bind $e2";
  exec.poststop+="jibe del  $e2";
````

### Point-to-point /31 wire connection

This formulation connects two devices (host-jail, parent-child jails)
via a CIDR/31 link. The specified address is assigned to the named
interface and the corresponding address (the other address in the /31
pair) is assigned to the other end of the epair. In addition, the
default gateway is set to the remote end of the epair.

````
/etc/jail.conf.d/p2p.conf:
  $e3 = "e3a_${name},172.31.39.23/31,d";

  vnet.interface=e3a_${name}

  exec.prestart+="jibe pre  $e3";
  exec.start   +="jibe bind $e3";
  exec.poststop+="jibe del  $e3";
````

Up until this example we've been using e#b_<jail> as the interface
passed into the jail. Here, as an example we use `e3a_p2p` (the 'a'
side) to be passed into the jail. In this example, `e3b_p2p` will be
assigned `172.31.39.22/31` and left with the device creating jail p2p.


### Specify Multiple Interfaces

This formulation combines the first two examples (DHCP and Static IP)
into a configuration that could be for a network device (like a
router). Here, we presume the physical network interface has been
added to `pub_bridge`, therefore, `e0a_router` has access to it and
receives a DHCP address. Meanwhile, other jails connect to 
`priv_bridge` and use this router as a gateway.

````
/etc/jail.conf.d/router.conf:

  $e0 = "e0b_${name},dhcp,02:92:71:65:e1:0b,b/pub_bridge";
  $e1 = "e1b_${name},172.31.40.1/24,b/priv_bridge";

  vnet.interface=e0b_${name} e1b_${name}

  exec.prestart+="jibe pre  $e0 $e1";
  exec.start   +="jibe bind $e0 $e1";
  exec.poststop+="jibe del  $e0 $e1";
````