---
title: "Practica"
date: 2022-12-13T13:13:40+01:00
draft: true
---

Tabla del contenido del dns

|maquina|V. Interna|V. Externa|V. DMZ|
|:-:|:-:|:-:|:-:|
|alfa|192.168.0.1|172.22.X.X|172.16.0.1|
|bravo|172.16.0.200|---|172.16.0.200|
|charlie|192.168.0.2|---|192.168.0.2|
|delta|192.168.0.3|---|192.168.0.3|
|www|bravo|alfa|bravo|
|bd|delta|---|delta|
|-|-|-|-|
|0.168.192.in-addr.arpa|Si|No|No|
|16.172.in-addr.arpa|Si|No|Si|
||↓|↓|↓|
||NS charlie|NS alfa|NS charlie|
