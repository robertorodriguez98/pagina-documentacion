---
title: "Second Content"
date: 2022-12-07T13:02:45+01:00
draft: true
---

##### 1. Indica la instrucción `virt-clone` que has usado para clonar la máquina. ¿Qué cambios has hecho en la nueva máquina para que no sea igual a la original?

```bash
virt-clone --connect=qemu:///system --original debianEjercicios \
--name maquina-clonada \
--file /media/roberto/Extraible/pool/maquina-clonada.qcow2
```

Le he puesto un nombre nuevo, además de indicar cuál va a ser el fichero que va a utilizar.

![t5-1](https://i.imgur.com/WHu7KPr.png)

##### 2. Lista las máquinas que tienes creada, donde se vea la plantilla que has creado. Pon una captura donde se vea que nos da un error al intentar iniciarla

![t5-2](https://i.imgur.com/txFWVsB.png)

![t5-3](https://i.imgur.com/13Arfr1.png)

##### 3. Una vez creada la máquina **clone-full**, una captura de pantalla donde se vea la dirección IP que ha tomado. Otra captura donde se vea el acceso por SSH

![t5-4](https://i.imgur.com/be4Y2kj.png)

![t5-5](https://i.imgur.com/S6hCUvL.png)

##### 4. Una vez realizada la clonación enlazada: La lista de máquinas donde se vea la máquina clone-link. La salida del comando `virsh -c qemu:///system vol-info <fichero.qcow2> <pool de almacenamiento>` para comprobar que el volumen creado tiene un disco de **Backing Store**. Comprueba lo que ocupa el disco creado. Debe ocupar muy poco en disco. ¿Por qué?

![t5-6](https://i.imgur.com/iEJH1Jm.png)

![t5-7](https://i.imgur.com/5J4P05R.png)

Ocupa 7 MiB ya que en el disco de la máquina con clonación enlazada, solo aparecen los cambios en el sistema de archivos.

##### 5. Muestra con capturas de pantallas el funcionamiento del ejercicio 5. Una vez creada la instantánea entrega la salida del comando `virsh -c qemu:///system snapshot-list <máquina>`

![t5-8](https://i.imgur.com/bNxqlKu.png)