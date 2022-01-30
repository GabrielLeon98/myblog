---
layout: post
title:  "Introducción a Cisco"
banner: "/assets/images/banners/Cisco-portada.png"
date:   2022-01-30 20:41:00 +0200
categories: Cisco
---

A partir de aquí empezaré una serie de post realizando configuraciones de switches y routers de Cisco. Mi idea es subir el contendo sin seguir un orden concreto, ya que no es mi intención hacer una especie de curso intensivo para preparar el CCNA ni nada de eso, por lo que tampoco entraré en aspectos teóricos básicos a nivel de redes (salvo alguna excepción). Lo que pretendo es subir distintos "laboratorios" que podeís encontrar en muchos sitios, aunque mi objetivo es explicarlos de tal manera que queden lo más claro posible.

Para los laboratoios utilizaré EVE-NG, ya que es el software de simulación de redes que más me gusta. Este software viene incluido en una máquina virtual la cual tendréis que iniciar cada vez que queráis hacer uso de la herramienta. Podeís descargar EVE-NG desde [aquí](https://www.eve-ng.net/). Para acceder, como he dicho, tenéis que iniciar la máquina virtual. Esta contiene un sistema operativo GNU/LINUX (Ubuntu, en este caso). Como recomendación, deberiais darle un direccionamiento estático, ya que la manera de acceder a EVE-NG una vez encendida la máquina virtual es a través del navegador web escribriendo la dirección IP de la máquina virtual (por lo que no queremos que esta cambien en algún momento dado).

El procedimiento para darle un direccionamiento estático lo explico a continuación:

Al iniciar la máquina virtual nos pide introducir un nombre de usuario y además, si nos fijamos, nos da la contraseña para el root y la dirección IP. El idioma del sistema operativo está en ingles, por lo que el teclado también estará en inglés, lo que significa que a la hora de ejecutar algunos comandos tendrémos que estar buscando cual es el caracter que queremos introducir. La forma más rápida de solucionar esto es haciendo uso de un cliente SSH, ya que de esta manera si tendremos el idioma del teclado al español. Si tenéis Windows, podeis usar PUTTY. Para ello, copiamos la dirección IP que nos facilita la máquina virtual y accedemos vía SSH.

Entraremos con el usuario root y editaremos el fichero ````/etc/network/interfaces````. Veremos que hay varias interfaces. Nosotros en este caso tenemos que editar la interfaz ````pnet0```` donde introduciremos:

* Dirección IP
* Máscara
* Puerta de enlace 

Quedando de la siguiente manera (los datos que aparecen son de ejemplo):

````console
auto pnet0
iface pnet0 inet static
    address 192.168.100.10
    netmask 255.255.255.0
    gateway 192.168.100.254
    bridge_ports eth0
    bridge_stp off
````

Las líneas ```` bridge_ports eth0 ```` y ````bridge_stp off ```` dejarlas tal y como viene por defecto.

Tras ello, reiniciamos el servicio de red ejecutando ````systemctl restart networking````. Para comprobar que los datos han sido introducidos correctamente, ejecutamos ````ip a | grep pnet0```` y nos saldrá en pantalla la configuración recién añadida:

````console
root@eve-vm:~# ip a | grep pnet0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master pnet0 state UP group default qlen 1000
13: pnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.100.10/24 brd 192.168.100.255 scope global pnet0
root@eve-vm:~#
````

Ahora que hemos comprobado que todo está OK, se puede hacer uso de EVE-NG introduciendo desde algún navegador web la IP que hemos asignado a la máquina virtual:

IMAGEN

Ahora que hemos realizado los preparativos para posteriormente hacer los laboratorios, es hora de entrar en materia con Cisco. Antes de empezar a hablar de configuraciones, lo primero es hablar de lo que es el dispositivo físicamente. 

La mayoría de dispositivos Cisco poseen tres tipos de puertos:

* Puerto de consola
* Puerto auxiliar
* Puertos Ethernet

Los puertos Ethernet son a los que conectamos los equipos de la red, el puerto de consola, al que nos conectamos con el llamado cable "Rollover" o cable de consola. Este último es el cable que se usa para realizar las primeras configuraciones en un dispositivo Cisco, ya sea un router o un switch, es decir, cuando el equipo no tiene ninguna configuración IP ni nada para poder conectarnos remotamente. El puerto auxiliar es para acceder a la CLI mediante un módem.

Supongamos que tenemos un router Cisco que no ha sido usado previamente, por lo que no tiene configuración alguna. Lo primero es conectarnos mediante un cable de consola al router. El cable de consola contiene en un extremo un conector RJ-45 y en el otro un conector puerto serie DB9 hembra. Los PC de hoy en día ya no traen consigo puerto serie, por lo que si no disponemos de un ordenador que tenga puerto serie, existen adaptadores de puerto serie a USB que podemos comprar a un precio razonable.

Una vez que tengamos conectado el cable de consola desde nuestro equipo al router accedemos a la CLI. En mi caso usaré PUTTY.


IMAGEN


Se ha de seleccionar el puerto COM adecuado, la velocidad en 9600.
