---
title: "Despliegue Python"
date: 2023-01-10T09:18:40+01:00
draft: true
---

Enunciado:

Vamos a desplegar la aplicación del tutorial de django. Como entorno de desarrollo tienes dos opciones:

Que tu entorno de desarrollo se la máquina bravo de tu entorno de desarrollo. Opción que dará más puntos.
Que tu entorno de desarrollo sea una máquina de openstack con el sistema operativo que quieras. Opción que dará menos puntos.
Vamos a configurar tu equipo como entorno de desarrollo para trabajar con la aplicación, para ello:

Realiza un fork del repositorio de GitHub: https://github.com/josedom24/django_tutorial.
Crea un entorno virtual de python3 e instala las dependencias necesarias para que funcione el proyecto.
Comprueba que vamos a trabajar con una base de datos sqlite. ¿Qué fichero tienes que consultar?. ¿Cómo se llama la base de datos que vamos a crear?
Crea la base de datos. A partir del modelo de datos se crean las tablas de la base de datos.
Crea un usuario administrador.
Ejecuta el servidor web de desarrollo y entra en la zona de administración (\admin) para comprobar que los datos se han añadido correctamente.
Crea dos preguntas, con posibles respuestas.
Comprueba en el navegador que la aplicación está funcionando, accede a la url \polls.
Configura el servidor web apache2 con el módulo wsgi para servir la página web. Si utilizas como entorno de desarrollo, la máquina bravo se acceder con el nombre poython.tunombre.gonzalonazareno.org. Si tu entorno de desarrollo es una máquina de openstack, elige el nombre con el que acceder y entrega la dirección IP de la máquina.

{{< hint type=tip title=Enunciado >}}
1. Realiza un fork del repositorio de GitHub: https://github.com/josedom24/django_tutorial.
2. Crea un entorno virtual de python3 e instala las dependencias necesarias para que funcione el proyecto.
3. Comprueba que vamos a trabajar con una base de datos sqlite. ¿Qué fichero tienes que consultar?. ¿Cómo se llama la base de datos que vamos a crear?
4. Crea la base de datos. A partir del modelo de datos se crean las tablas de la base de datos.
5. Crea un usuario administrador.
{{< /hint >}}

| Marca de control | Resultado de **éxito** del módulo                                                                                                                             | Resultado de **fracaso** del módulo                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| requisite        | La pila continua ejecutándose y puede fracasar o tener éxito dependiendo del resultado de otros módulos                                                       | La pila deja de ejecutarse,fracaso                                                                                                                  |
| required         | la pila continua ejecutándose y puede fracasar o tener éxito dependiendo del resultado de otros módulos                                                       | La pila continua ejecutandose pero fracasa en última instancia                                                                                      |
| sufficient       | La pila deja de ejecutarse y tiene éxito a no ser que un módulo requisite o required anterior fracasara                                                       | La pila continua ejecutándose y puede fracasar o tener éxito dependiendo del resultado de otros módulos                                             |
| optional         | La pila continua ejecutándose y puede fracasar o tener éxito dependiendo del resultado de otros módulos (si es el único módulo de la pila, ésta tiene éxito.) | La pila continua ejecutándose y puede fracasar o tener éxito dependiendo del resultado de otros módulos ( si es el único módulo de la pila fracasa) |