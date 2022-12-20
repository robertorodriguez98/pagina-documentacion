---
title: "Escenario Dns"
date: 2022-12-19T14:01:03+01:00
draft: true
---

#### VISTAS

```bash
view interna {
    match-clients { 192.168.0.0/24; 127.0.0.1; };
    allow-recursion { any; };
        zone "roberto.gonzalonazareno.org"
        {
        type master;
        file "db.interna.roberto.gonzalonazareno.org";
        };
        zone "0.168.192.in-addr.arpa"
        {
        type master;
        file "db.0.168.192";
        };
        zone "16.172.in-addr.arpa"
        {
        type master;
        file "db.0.16.172";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
    };

view dmz {
    match-clients { 172.16.0/16;};
    allow-recursion { any; };
        zone "roberto.gonzalonazareno.org"
        {
        type master;
        file "db.dmz.roberto.gonzalonazareno.org";
        };
        zone "16.172.in-addr.arpa"
        {
        type master;
        file "db.16.172";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
    };

view externa {
    match-clients { 172.22.0.0/16; 172.29.0.0/16; 192.168.202.2; };
    allow-recursion { any; };
        zone "roberto.gonzalonazareno.org"
        {
        type master;
        file "db.externa.roberto.gonzalonazareno.org";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};
```

## Definición de las zonas

### INTERNA /var/cache/bind/db.interna.roberto.gonzalonazareno.org

```bash
$TTL    86400
@       IN      SOA     charlie.roberto.gonzalonazareno.org. root.roberto.gonzalonazareno.org. (
                            1           ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        86400 )         ; Negative Cache TTL
;
@	IN	NS		charlie.roberto.gonzalonazareno.org.
@	IN	MX	10	mail.roberto.gonzalonazareno.org.

$ORIGIN roberto.gonzalonazareno.org.

alfa        IN  A       192.168.0.1
bravo       IN  A       172.16.0.200
charlie     IN  A       192.168.0.2
delta       IN  A       192.168.0.3
www         IN  CNAME   bravo
bd          IN  CNAME   delta
```


### INTERNA INVERSA /var/cache/bind/db.0.168.192

```bash
$TTL    86400
@       IN      SOA     charlie.roberto.gonzalonazareno.org. root.roberto.gonzalonazareno.org. (
                            1         ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        86400 )       ; Negative Cache TTL
;
@	IN	NS		charlie.roberto.gonzalonazareno.org.

$ORIGIN 0.168.192.in-addr.arpa.

1			IN	PTR		alfa.roberto.gonzalonazareno.org.
2            IN	PTR		charlie.roberto.gonzalonazareno.org.
3            IN	PTR		delta.roberto.gonzalonazareno.org.
```

### INTERNA INVERSA /var/cache/bind/db.16.172

```bash
$TTL    86400
@       IN      SOA     charlie.roberto.gonzalonazareno.org. root.roberto.gonzalonazareno.org. (
                            1         ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        86400 )       ; Negative Cache TTL
;
@	IN	NS		charlie.roberto.gonzalonazareno.org.

$ORIGIN 16.172.in-addr.arpa.

1.0			IN	PTR		alfa.roberto.gonzalonazareno.org.
200.0            IN	PTR		bravo.roberto.gonzalonazareno.org.
```

### DMZ /var/cache/bind/db.dmz.roberto.gonzalonazareno.org

```bash
$TTL    86400
@       IN      SOA     charlie.roberto.gonzalonazareno.org. root.roberto.gonzalonazareno.org. (
                            1           ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        86400 )         ; Negative Cache TTL
;
@	IN	NS		charlie.roberto.gonzalonazareno.org.

$ORIGIN roberto.gonzalonazareno.org.

alfa        IN  A       172.16.0.1
bravo       IN  A       172.16.0.200
charlie     IN  A       192.168.0.2
delta       IN  A       192.168.0.3
www         IN  CNAME   bravo
bd          IN  CNAME   delta
```

### EXTERNA /var/cache/bind/db.externa.roberto.gonzalonazareno.org

```bash
$TTL    86400
@       IN      SOA     alfa.roberto.gonzalonazareno.org. root.roberto.gonzalonazareno.org. (
                            1           ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        86400 )         ; Negative Cache TTL
;
@	IN	NS		alfa.roberto.gonzalonazareno.org.

$ORIGIN roberto.gonzalonazareno.org.

alfa        IN  A       172.22.200.218
www         IN  CNAME   alfa
```
