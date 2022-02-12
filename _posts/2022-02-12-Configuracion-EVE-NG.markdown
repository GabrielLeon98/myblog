---
layout: post
title:  "Configuración de EVE-NG"
banner: "/assets/images/banners/eve-ng.png"
date:   2022-02-12 10:00:00
categories: EVE-NG
---

Para los laboratorios utilizaré EVE-NG, ya que es el software de simulación de redes que más me gusta. Este software viene incluido en una máquina virtual, la cual se encargará de ejecutar a su vez todas las imágenes de los sistemas operativos de los dispositivos que iniciemos durante un laboratorio. Podeís descargar EVE-NG desde aquí. Para acceder se tiene que iniciar la máquina virtual, la cual contiene un sistema operativo GNU/LINUX (Ubuntu, en este caso). Como recomendación, deberiais darle un direccionamiento estático, ya que la manera de acceder a EVE-NG una vez encendida la máquina virtual es a través del navegador web escribriendo la dirección IP de la máquina virtual (por lo que no queremos que esta cambie en algún momento dado).

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

![eve](https://i.ibb.co/qN222P1/introduccion-a-cisco-1.jpg "EVE-NG")

Las credenciales para acceder son:

* Usuario: admin
* Clave: eve

Tras ello accedemos a la interfaz de EVE-NG donde a continuación explico un poco como funciona:
