---
title: "Resolución de problemas de
tablas mutantes"
date: 2022-12-19T00:51:41+01:00
draft: false
---

El enunciado a resolver y que contiene una tabla mutante es el siguiente:

6 - Realiza los módulos de programación necesarios para evitar que un catador puntue más de tres aspectos de una misma versión de un experimento.


En este caso, el trigger principal sería sobre la tabla puntuaciones, y dentro, para comprobar que se cumple la condición de que un catador no puntúa más de 3 aspectos, se tendría que hacer una consulta a la misma tabla puntuaciones, que en ese caso estaría mutando. Para resolver el problema vamos a realizar los siguientes pasos:

1. Leer bien los requisitos del problema e identificar la
información de la tabla que debo guardar en variables
persistentes.
1. Crear el paquete declarando los tipos de datos y las variables
necesarias para guardar dicha información.
1. Hacer un trigger before por sentencia que rellene dichas
variables consultando la tabla mutante.
1. Hacer un trigger before por fila que compruebe si el registro
que se está manejando cumple la condición especificada
consultando las variables persistentes.
1. En muchas ocasiones, el trigger por fila también tendrá que ir
actualizando la información que está en las variables

### 1. Leer bien los requisitos del problema e identificar la información de la tabla que debo guardar en variables persistentes

El contenido de la tabla de puntuaciones es el siguiente:

![Tabla puntuaciones](https://i.imgur.com/wMxNzMj.png)

En este caso, tendríamos que almacenar en variables persistentes los datos de la consulta, que en este caso son los siguientes campos:

* NIFCatador
* CodigoAspecto
* CodigoExperimento
* CodigoVersion

### 2. Crear el paquete declarando los tipos de datos y las variables necesarias para guardar dicha información

```sql
CREATE OR REPLACE PACKAGE ControlPuntuaciones
AS
TYPE tRegistroTablaPuntuaciones IS RECORD --defino el tipo de datos registro
(
NIFCatador Puntuaciones.NIFCatador%TYPE,
CodigoAspecto Puntuaciones.CodigoAspecto%TYPE,
CodigoExperimento Puntuaciones.CodigoExperimento%TYPE,
CodigoVersion Puntuaciones.CodigoVersion%TYPE
);

TYPE tTablasPuntuaciones IS TABLE OF tRegistroTablaPuntuaciones -- defino el tipo de datos tabla
INDEX BY BINARY_INTEGER;
PuntuacionesCatador tTablasPuntuaciones;
-- declaro una variable del tipo tabla antes creado
END ControlPuntuaciones;
/
```

### 3. Hacer un trigger before por sentencia que rellene dichas variables consultando la tabla mutante

```sql
CREATE OR REPLACE TRIGGER RELLENARPUNTUACIONES
BEFORE INSERT OR UPDATE ON Puntuaciones
DECLARE
CURSOR c_puntuaciones IS SELECT NIFCatador,CodigoAspecto,CodigoExperimento,CodigoVersion
FROM Puntuaciones;
INDICE NUMBER:=0;
BEGIN
-- relleno la tabla de sueldos de los jefes de cada departamento
FOR v_puntuacion IN c_puntuaciones LOOP
ControlPuntuaciones.PuntuacionesCatador(INDICE).NIFCatador := v_puntuacion.NIFCatador;
ControlPuntuaciones.PuntuacionesCatador(INDICE).CodigoAspecto := v_puntuacion.CodigoAspecto;
ControlPuntuaciones.PuntuacionesCatador(INDICE).CodigoExperimento := v_puntuacion.CodigoExperimento;
ControlPuntuaciones.PuntuacionesCatador(INDICE).CodigoVersion := v_puntuacion.CodigoVersion;
INDICE := INDICE + 1;
END LOOP;
END RELLENARPUNTUACIONES;
/
```

### 4. Hacer un trigger before por fila que compruebe si el registro que se está manejando cumple la condición especificada consultando las variables persistentes

```sql
CREATE OR REPLACE TRIGGER ControlarPuntuaciones
BEFORE INSERT OR UPDATE ON Puntuaciones
FOR EACH ROW
DECLARE
BEGIN
-- compruebo que el catador no haya puntuado más de 3 aspectos
IF (NumeroPuntuacionesCatador(:NEW.NIFCatador,:NEW.CodigoExperimento,:NEW.CodigoVersion) > 3) THEN
RAISE_APPLICATION_ERROR(-20001,'El catador no puede puntuar más de 3 aspectos');
end if;
END;
/
```

Voy a crear una funcion para comprobar el numero de puntuaciones que tiene un catador en una versión de un experimento:

```sql
CREATE OR REPLACE FUNCTION NumeroPuntuacionesCatador
( p_NIFCatador Puntuaciones.NIFCatador%TYPE, p_CodigoExperimento Puntuaciones.CodigoExperimento%TYPE, p_CodigoVersion Puntuaciones.CodigoVersion%TYPE)
RETURN NUMBER
IS
    v_NumeroPuntuaciones NUMBER:= 0;
    v_cantidad NUMBER;
BEGIN
    for i in ControlPuntuaciones.PuntuacionesCatador.FIRST..ControlPuntuaciones.PuntuacionesCatador.LAST
    LOOP
        if (ControlPuntuaciones.PuntuacionesCatador(i).NIFCatador = p_NIFCatador) and (ControlPuntuaciones.PuntuacionesCatador(i).CodigoExperimento = p_CodigoExperimento) and (ControlPuntuaciones.PuntuacionesCatador(i).CodigoVersion = p_CodigoVersion) then
            v_NumeroPuntuaciones := v_NumeroPuntuaciones + 1;
        end if;
    END LOOP;
    return v_NumeroPuntuaciones;
end;
/

--- prueba

create or replace procedure prueba
is
numero number;
begin
numero:=NumeroPuntuacionesCatador('14425879A','A0003-A','0.0.2');
dbms_output.put_line(numero);
end;
/
insert into puntuaciones values('14425879A','0004','A0003-A','0.0.2',7.5);


create or replace procedure imprimirtabla
is
begin
    for i in ControlPuntuaciones.PuntuacionesCatador.FIRST..ControlPuntuaciones.PuntuacionesCatador.LAST
    LOOP
        dbms_output.put_line('Registro numero '|| i);
        dbms_output.put_line('NIFCatador: '|| ControlPuntuaciones.PuntuacionesCatador(i).NIFCatador);
        dbms_output.put_line('CodigoAspecto: '|| ControlPuntuaciones.PuntuacionesCatador(i).CodigoAspecto);
        dbms_output.put_line('CodigoExperimento: '|| ControlPuntuaciones.PuntuacionesCatador(i).CodigoExperimento);
        dbms_output.put_line('CodigoVersion: '|| ControlPuntuaciones.PuntuacionesCatador(i).CodigoVersion);


    end loop;
end;
/
```

### 5. En muchas ocasiones, el trigger por fila también tendrá que ir actualizando la información que está en las variables

```sql
insert into puntuaciones values('14425879A','0004','A0003-A','0.0.2',7.5);
insert into puntuaciones values('14425879A','0005','A0003-A','0.0.2',7.5);
insert into puntuaciones values('14425879A','0006','A0003-A','0.0.2',7.5);
insert into puntuaciones values('14425879A','0007','A0003-A','0.0.2',7.5);
```