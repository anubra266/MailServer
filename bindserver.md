### Install bind
```
apt-get install bind9 bind9utils bind9-doc
```

### Open the main configuration file using the command:
```
nano /etc/bind/named.conf
```

### And make sure that the following lines are included in the file:
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

### Now we need to define our DNS zone. To do so, open the file `/etc/bind/named.conf.local`:
```
nano /etc/bind/named.conf.local

```
### And insert the following to the file:
```
zone "anubra.tech" {
        type master;
        file "/etc/bind/anubra.tech";
 };
```

Keep in mind that `anubra.tech` should be replaced with your own domain name.

It will tell BIND9 to look for the file /etc/bind/anubra.tech to find the DNS zone for anubra.tech.

### Now you must open the zone file and add the necessary DNS records there:
```
nano /etc/bind/anubra.tech
```

The following is an example zone file:

```
$TTL 3600
@	IN	SOA	ns1.anubra.tech. root.anubra.tech. (
			2020062710
			3600
			1800
			604800
			86400 )
@       IN  NS          ns1.anubra.tech.
@       IN  NS          ns2.anubra.tech.
ns1     IN  A           199.192.22.73
ns2     IN  A           199.192.22.73
@       IN  A           199.192.22.73
www     IN  A           199.192.22.73

# Wildcard Subdomain Routing

*       IN  A           199.192.22.73
```