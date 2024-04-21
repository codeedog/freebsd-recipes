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
# /etc/jail.conf.d/jail_dhcp.conf

  $e0 = "e0b_${name},dhcp,b/pub_bridge";

  # Network
  vnet.interface=e0b_${name}

  exec.prestart+="jibe pre  $e0";
  exec.start   +="jibe bind $e0";
  exec.poststop+="jibe del  $e0";
  exec.prestart+="ifconfig pub_bridge addm eth0 up";
````



    echo "$pgm <pre|bind|del> if_desc1[ if_desc2 [ if_desc3 ...]]"                                                    
    echo "  if_desc# := e#<a|b>[_name][,dhcp|inet <IPv4/CIDR>][,b/<bridge_name>][,<mac_address>][,d[=<IPv4|if>]]"     
