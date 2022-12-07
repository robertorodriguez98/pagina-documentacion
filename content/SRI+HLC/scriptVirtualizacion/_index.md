---
title: "Script virtualización"
date: 2022-12-07T18:26:16+01:00
draft: false
---

##### 1. Entrega la URL del repositorio GitHub donde has alojado el proyecto.

[https://github.com/robertorodriguez98/script-virtualizacion](https://github.com/robertorodriguez98/script-virtualizacion)
##### 2. Indica los pasos que has realizado para la creación de la imagen base.

Los pasos han sido los siguientes:

1. he creado una máquina virtual con los siguientes parámetros:

```bash
virt-install --connect qemu:///system \                 
                         --virt-type kvm \
                         --name bullseye-base \
                         --cdrom ~/Descargas/firmware-11.5.0-amd64-netinst.iso \
                         --os-variant debian10 \
                         --disk size=3 \
                         --memory 1024 \
                         --vcpus 1
```

2. Durante la instalación, he especificado que se use xfs, y que el usuario sea debian con contraseña debian y pertenezca al grupo sudo.
3. Una vez finalizada la instalación añadimos al fichero `/etc/sudoers` la siguiente línea:

```bash
debian    ALL=NOPASSWD: ALL
```

4. En nuestra máquina, creamos el par de claves:

```bash
ssh-keygen -t ecdsa -f clave-ecdsa
```

5. Por último, reducimos la imagen::

```bash
sudo virt-sparsify /var/lib/libvirt/images/bullseye-base.qcow2 bullseye-base.qcow2
```

##### 3. Entrega la clave privada que has utilizado y un enlace para descargarme la imagen base.

La clave privada es la siguiente:

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQTOfJV8eqrR37bfuBdchdft3g9gCS+W
V/tLbKrD5rSdtzVyTrH4YnsPSCNsTcMZ4FwQz8buz9xlhsw/6OEzsfkEAAAAsLoD6Yq6A+
mKAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBM58lXx6qtHftt+4
F1yF1+3eD2AJL5ZX+0tsqsPmtJ23NXJOsfhiew9II2xNwxngXBDPxu7P3GWGzD/o4TOx+Q
QAAAAhAP3sLtPBkJ0p7v/n+KSUl9jkx/pXqUFtwKbJgXGYz2+5AAAAEHJvYmVydG9AcG9y
dGF0aWwBAgMEBQYH
-----END OPENSSH PRIVATE KEY-----
```

Y la imagen base está alojada en el siguiente enlace:

[https://mega.nz/file/i8liCSaY#tC8XL4jh4sLhlnBJs4yey0zQTA6z1BK9JGm1bowmmmk](https://mega.nz/file/i8liCSaY#tC8XL4jh4sLhlnBJs4yey0zQTA6z1BK9JGm1bowmmmk)

##### 4. Ejecuta el script y cuando se pause. Entrega pantallazo donde se compruebe que se puede acceder al servidor web en la maquina1.

![p1-1](https://i.imgur.com/hiVQzWN.png)

##### 5. Al finalizar el script: pantallazo donde se compruebe que se puede acceder al servidor web con la IP pública.

![p1-2](https://i.imgur.com/oOO634g.png)

##### 6. Al finalizar el script: Pantalalzos para comprobar:

* Que la máquina tiene montado un disco en el directorio /var/www/html.
![p1-3](https://i.imgur.com/RPcHE3f.png)
* Que la máquina tiene 2G de RAM.
![p1-4](https://i.imgur.com/d5wLOPR.png)
* Que accediendo a la máquina puedes acceder al contenedor.
![p1-5](https://i.imgur.com/yPjQdhy.png)
* Que se ha ha creado un snapshot.
![p1-6](https://i.imgur.com/YwGtOiF.png)
