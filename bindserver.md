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