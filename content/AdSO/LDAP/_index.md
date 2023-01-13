---
title: "LDAP"
date: 2023-01-12T13:14:30+01:00
draft: true
---

### Requisitos
* Hay que tener un fqdn
* dns configurado
* Los relojes de las máquinas tienen que estar sincronizados

```bash
apt show slapd
apt install slapd
dpkg-reconfigure --priority=low slapd
```

Comprobamos que está funcionando

```bash
ss -lntp
systemctl status slapd
```

![foto](ldapesquema.png)

Para ver los binarios del paquete:

```bash
dpkg -L slapd | egrep bin
```

```bash
sudo slapcat
```

para ver el esquema:

```bash
$ sudo tree /etc/ldap/slapd.d/

/etc/ldap f
```