---
title: "Integridad y autenticidad"
date: 2022-12-13T11:00:32+01:00
draft: false
---

#### 1. Manda un documento y la firma electrónica del mismo a un compañero. Verifica la firma que tu has recibido

Creamos los fichero:

```bash
echo "DOCUMENTO CIFRADO SECRETO" > documento_rober.txt
gpg --output firmarober.sig --detach-sig documento_rober.txt
```

Podemos verficar la la firma del documento con

```bash
gpg --verify firma_nazareth.sig secreto_nazareth.txt
```

![1](https://i.imgur.com/d3KcJjy.png)

#### 2. ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

```bash
 gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
 gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
 gpg:          No hay indicios de que la firma pertenezca al propietario.
 Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC
```

Significa que la firma es correcta, pero que no hay indicios de que la firma pertenezca al propietario, por lo que no es de confianza.

#### 3. Vamos a crear un anillo de confianza entre los miembros de nuestra clase, para ello

* Tu clave pública debe estar en un servidor de claves
* Escribe tu fingerprint en un papel y dárselo a tu compañero, para que puede descargarse tu clave pública.
* Te debes bajar al menos tres claves públicas de compañeros. Firma estas claves.
* Tu te debes asegurar que tu clave pública es firmada por al menos tres compañeros de la clase.
* Una vez que firmes una clave se la tendrás que devolver a su dueño, para que otra persona se la firme.
* Cuando tengas las tres firmas sube la clave al servidor de claves y rellena tus datos en la tabla Claves públicas PGP 2020-2021
* Asegurate que te vuelves a bajar las claves públicas de tus compañeros que tengan las tres firmas.

#### 4. Muestra las firmas que tiene tu clave pública

Mi clave publica tiene las siguientes firmas:

![2](https://i.imgur.com/I59ISzh.png)

#### 5. Comprueba que ya puedes verificar sin “problemas” una firma recibida por una persona en la que confías

![3](https://i.imgur.com/VTptr3p.png)

#### 6. Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total

