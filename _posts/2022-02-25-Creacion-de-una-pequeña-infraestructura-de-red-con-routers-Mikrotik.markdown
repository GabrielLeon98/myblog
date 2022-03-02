---
layout: post
title:  "Creación de una pequeña infraestructura de red con routers Mikrotik"
banner: "/assets/images/banners/mikrotik-logo.jpg"
date:   2022-02-25 9:00:00 +0200
categories: Mikrotik
---

En este post voy a mostrar la configuración de una pequeña infraestructura de una red la cual contiene los siguientes dispositivos:

* Dos routers Mikrotik.
* Dos equipos clientes con un Windows 7.
* Un equipo con un sistema operativo Linux que hará de servidor.
* Dos switches.

La infraestructura a nivel de conexiones está compuesta por:

* Dos redes LAN.
* Un enlace entre los dos routers que simula un entorno WAN.

El objetivo del post es mostar la configuración de los routers Mikrotik para tal escenario (enrutamiento, reglas de cortafuegos, DHCP, etc) y hacer pruebas que muestren su correcto funcionamiento. A continuación os muestro el esquema del escenario, que contiene las direcciones IP de los equipos y las distintas conexiones:

![escenario1](https://ibb.co/LSDf0RF) 
