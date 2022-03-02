---
layout: post
title:  "Creación de una pequeña infraestructura de red con routers Mikrotik"
banner: "/assets/images/banners/mikrotik-logo.jpg"
date:   2022-02-25 9:00:00 +0200
categories: Mikrotik
---

## Introducción

En este post voy a mostrar la configuración de una pequeña infraestructura de una red la cual contiene los siguientes dispositivos:

* Dos routers Mikrotik.
* Dos equipos clientes con un Windows 7.
* Un equipo con un sistema operativo Linux que hará de servidor.
* Dos switches.

La infraestructura a nivel de conexiones está compuesta por:

* Dos redes LAN.
* Un enlace entre los dos routers que simula un entorno WAN.

El objetivo del post es mostar la configuración de los routers Mikrotik para tal escenario (enrutamiento, reglas de cortafuegos, DHCP, etc) y hacer pruebas que muestren su correcto funcionamiento. A continuación os muestro el esquema del escenario, que contiene las direcciones IP de los equipos y las distintas conexiones:

![escenario1](https://i.ibb.co/BLHRf3p/escenario-1.jpg) 

Los equipos tendrán en primer lugar un direccionamiento estático hasta que más adelante muestre el proceso de configuración de un servidor DHCP en los routers. Las direcciones IP que pondremos en los equipos serán las mismas que vienen en el esquema.

Los routers Mikrotik pueden ser configurados o bien a través de un terminal mediante la ejecución de comandos o mediante entorno gráfico a través del software que nos proporciona el propio fabricante para su gestión llamado *Winbox* el cual podemos obtener desde la página web del fabricante. Explicaré la manera de realizar las distintas configuraciones de las dos formas ya que considero que tanto una como otra son útiles de saber. El software de simulación que voy a usar para representar este escenario es EVE-NG.

En la imagen se puede apreciar que he dado nombre a las dos redes locales: una con **SEDE SEVILLA** y otra con **SEDE BARCELONA**. La utilidad de esto es para representar lo que sería una red de una empresa y así, a la hora de explicarr que sea más fácil dirigirme a una u a otra.

Una vez dada esta pequeña introducción, empezemos pues con las configuraciones.

## Configuración de los routers y de los equipos

Vamos a empezar dando direcciones IP a los routers. Para ello nos conectaremos desde los equipos usando *Winbox* a los routers. Estos routers tienen habilitado por defecto una función llamada *MAC Server* la cual nos permite conectarnos a los routers teniendo en cuenta únicamente la MAC de la interfaz del router que está conectada al mismo switch al que estamos conectados. Esto es una brecha de seguridad puesto que de esta manera se permite que cualquier equipo aún teniendo una IP de otra subred pueda conectarse, por lo que esta opción la deshabilitaremos. Los routers Mikrotik suelen traer por defecto la dirección IP 192.168.88.1/24, aun que en este caso no la tienen, por lo que se la pondremos nosotros mismos.

Lo que haremos pues es conectarnos utilizando en este caso el *MAC Server* y luego le daremos una dirección IP a las interfaces respentando el direccionamiento que viene en el esquema:

### Configuración de los routers

Nos conectamos desde *Winbox*

![conexionrouter](https://i.ibb.co/ph1vzqt/mikrotik-1-1.jpg)


