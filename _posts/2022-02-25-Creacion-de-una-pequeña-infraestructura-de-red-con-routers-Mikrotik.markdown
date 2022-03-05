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
* Un equipo con un sistema operativo Linux que hará de servidor (contiene servidor web y SSH).
* Dos switches.

La infraestructura a su vez se encuentra dividida por:

* Dos redes LAN.
* Un enlace entre los dos routers que simula un entorno WAN.

El objetivo del post es mostar la configuración de los routers Mikrotik para tal escenario (asignación de direcciones IP a las interfaces, enrutamiento, reglas de cortafuegos, DHCP, etc) y hacer pruebas que muestren su correcto funcionamiento. A continuación os muestro el escenario, que contiene las direcciones IP de los equipos y las distintas conexiones:

![escenario1](https://i.ibb.co/BLHRf3p/escenario-1.jpg) 

Los equipos tendrán en primer lugar un direccionamiento estático hasta que más adelante muestre el proceso de configuración de un servidor DHCP en los routers. Las direcciones IP que pondremos en los equipos serán las mismas que vienen en el esquema.

Los routers Mikrotik pueden ser configurados o bien a través de un terminal mediante la ejecución de comandos o mediante entorno gráfico a través del software que nos proporciona el propio fabricante para su gestión llamado *Winbox* el cual podemos obtener desde la página web del fabricante. Explicaré la manera de realizar las distintas configuraciones de las dos formas ya que considero que tanto una como otra son útiles de saber. El software de simulación que voy a usar para representar este escenario es EVE-NG.

En la imagen se puede apreciar que he dado nombre a las dos redes locales: una con **SEDE SEVILLA** y otra con **SEDE BARCELONA**. La utilidad de esto es para representar lo que sería una red de una empresa y así, a la hora de explicarr que sea más fácil dirigirme a una u a otra.

Una vez dada esta pequeña introducción, empezemos pues con las configuraciones.

## Configuración de los routers y de los equipos

Vamos a empezar dando direcciones IP a los routers. Para ello nos conectaremos desde los equipos usando *Winbox* a los routers. Estos routers tienen habilitado por defecto una función llamada *MAC Server* la cual nos permite conectarnos a los routers teniendo en cuenta únicamente la MAC de la interfaz del router que está conectada al mismo switch al que estamos conectados, por lo que no se requiere que tanto el router como el equipo desde el que nos conectamos tengan una dirección IP de la misma subred. Esto es una brecha de seguridad puesto que de esta manera se permite que cualquier equipo aún teniendo una IP de otra subred pueda conectarse, por lo que esta opción la deshabilitaremos. Los routers Mikrotik suelen traer por defecto la dirección IP 192.168.88.1/24, aun que en este caso no la tienen pero podemos ponérselas si queremos.

Lo que haremos pues es conectarnos aprovechando el *MAC Server* y luego le daremos una dirección IP a las interfaces respentando el direccionamiento que viene en el esquema:

### Configuración de los routers

Los pasos los repetiremos en ambos routers. Primero nos conectamos desde *Winbox*. Al abrir el programa, en la pestaña *Neighbors* vemos como nos detecta al router. Para conectarnos debemos seleccionar la dirección MAC del dispositivo y el usuario por defecto es admin (la contraseña la configuraremos tras entrar al router, por lo que al principio no tiene):

![conexionrouter](https://i.ibb.co/ph1vzqt/mikrotik-1-1.jpg)

Insertamos una contraseña para el usuario *admin*:

![configcontraseña](https://i.ibb.co/RjFBLnN/mikrotik-1-2.jpg)

Como buena práctica, se recomienda dar un nombre concreto a los routers para que sea más fácil distinguirlos. Para ello vamos al menú de la izquierda en *Winbox* - *System* y vamos a *Identity* para introducir el nombre que identificará al router:

![identity](https://i.ibb.co/qnjdpPp/mikrotik-identity.jpg)

O bien desde comandos, abriendo la consola haciendo clic en *New Terminal* en el menú de la izquierda:
````terminal
[admin@Mikrotik] > system/identity/set name=Router-SE
[admin@Router-SE] >
````

Ahora lo que haremos será asignar las direcciones IP a las interfaces tal y como viene en el esquema.

Para asignar una IP a una interfaz concreta del Mikrotik, en el menú de la izquierda de *Winbox*, vamos a *IP* – *Addresess*. Aquí veremos las direcciones IP asignadas a cada interfaz. Para añadir una nueva IP hacemos clic en **+** donde se nos abrirá una ventana en donde tendremos que introducir: dirección IP y máscara en formato CIDR, dirección de subred (se pone automáticamente) e interfaz respectivamente.

Tras introducir correctamente los datos, hacemos click en *OK*:

* **Router-SE**:

![iprouterSE](https://i.ibb.co/kGB5Ltd/mikrotik-direccionamiento-SE.jpg) 

Comprobamos:

![iprouter1](https://i.ibb.co/cYLDxkN/mikrotik-1-3-2.jpg)

Si queremos realizar la misma tarea mediante comandos:
````terminal
[admin@Router-SE] > ip/address/add address=192.168.41.254/24 interface=ether2
````

Y comprobamos ejecutando:
````terminal
[admin@Router-SE] > ip/address/print
Columns: ADDRESS, NETWORK, INTERFACE
# ADDRESS            NETWORK       INTERFACE
0 192.168.21.254/24  192.168.21.0  ether1
[admin@Router-SE] >
````


* **Router-BACLN**:

![iprouter2](https://i.ibb.co/zhN4QpG/mikrotik-direccionamiento-BACLN.jpg)

Comprobamos:

![iprouter2](https://i.ibb.co/Gc1k4gC/mikrotik-1-3-1.jpg)

Si queremos realizar la misma tarea mediante comando:
````terminal
[admin@Router-BACLN] > ip/address/add address=192.168.41.254/24 interface=ether2
````

Y comprobamos ejecutando:
````terminal
[admin@Router-BACLN] > ip/address/print
Columns: ADDRESS, NETWORK, INTERFACE
# ADDRESS            NETWORK       INTERFACE
0 192.168.41.254/24  192.168.41.0  ether2
[admin@Router-BACLN] >
````

Podemos aprovechar para asignar a cada router la dirección IP para en enlace WAN. El procedimiento es el mismo:

* **Router-SE**:
![ipwanSE](https://i.ibb.co/4t9f8pM/wan-se.jpg)

* **Router-BACLN**:
![ipwanBACLN](https://i.ibb.co/qmLpdVt/wan-bacln.jpg)

Una vez introducidas las direcciones IP en los routers, vamos a deshabilitar el *MAC Server* y accederemos a ellos mediante las direcciones IP que hemos puesto (aun que antes tendremos que darle una dirección IP válida a los clinetes). Para deshabilitar el *MAC Server* vamos a *Tools* - *MAC Server* y hacemos clic en **MAC WinBox Server**. En *Allowed Interface List* seleccionamos **none**:

![macserver](https://i.ibb.co/fXRKc4s/mac-server.jpg)

O bien desde la terminal:
````terminal
[admin@Router-SE] > tool/mac-server/mac-winbox/set allowed-interface-list=none
````

### Configuración de los clientes

Ahora para acceder a *Winbox* debemos configurar en los clientes una dirección IP de la misma subred. Pondremos las mismas que aparecen en el esquema.

* **Cliente 1**:

![ipcliente1](https://i.ibb.co/THYSYhv/cliente-1.jpg)

* **Cliente 2**:

![ipcliente2](https://i.ibb.co/PtKcQLD/cliente-2.jpg)

* **Servidor**:

El servidor usa una distribución Ubuntu Mate. Para configurar una dirección IP desde la terminal como *root* realizamos los siguientes pasos:

Abrimos el fichero ````/etc/network/interfaces````:

![serv1](https://i.ibb.co/Z1Mr2Br/serv1.jpg)

Introducimos las siguientes líneas:

````terminal
auto eth0
iface eth0 inet static
address 192.168.41.200
netmask 255.255.255.0
network 192.168.41.0
gateway 192.168.41.254
````

Por último reinicamos el servicio de red ejecutando ````/etc/init.d/networking restart````.

Si se observa las capturas anteriores, tanto en los clientes como en el servidor hemos aprovechado para poner las puertas de enlace para que posteriormente podamos enviar paquetes de una subred a otra.

## Enrutamiento estático

Desde **Cliente 2** podemos ver la página del servidor web simplemente introduciendo la dirección IP de este en un navegador web:

![servidor1](https://i.ibb.co/kXBRRC1/servidor-1.jpg)

De igual manera podemos acceder remotamente a través de SSH: 

![servidor2](https://i.ibb.co/DQqWSjn/servidor-2.jpg)

Si queremos hacer lo mismo pero desde **Cliente 1**, debemos configurar algún enrutamiento, es decir, debemos indicar a los routers que camino tomar para llegar a una subred de destino. Como solamente son dos routers, configuraremos un enrutamiento estático.

### Comprobación de conectividad

Antes que nada vamos a comprobar que el enlace está bien configurado. Para ello vamos a hacer ping desde uno de los routers. Podemos hacerlo o bien desde *Winbox* en *Tools* - *Ping*:

![ping1](https://i.ibb.co/02t33C5/ping-1.jpg)

O desde el terminal:
````terminal
[admin@Router-SE] > ping 1.1.1.41
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 1.1.1.41                                   56  64 623us
    1 1.1.1.41                                   56  64 626us
    2 1.1.1.41                                   56  64 693us
    3 1.1.1.41                                   56  64 1ms198us
    sent=4 received=4 packet-loss=0% min-rtt=623us avg-rtt=785us
   max-rtt=1ms198us

[admin@Router-SE] >
````

### Creación de la entrada estática en la tabla de rutas

Una vez comprobado que hay conectividad vamos a crear las entradas estáticas en la tabla de rutas de ambos routers. La entrada ha de crearse en ambos ya que los dos deben conocer las subredes de destino para que se puedan enviar y recibir paquetes.

Para ello, desde *Winbox* vamos a *IP* - *Routes*:

![routelistSE](https://i.ibb.co/99dTwPV/route-list.jpg)

O desde el terminal:
````terminal
[admin@Router-SE] > ip/route/print
Flags: D - DYNAMIC; A - ACTIVE; c, y - COPY
Columns: DST-ADDRESS, GATEWAY, DISTANCE
    DST-ADDRESS      GATEWAY  DISTANCE
DAc 1.1.1.0/24       ether2          0
DAc 192.168.21.0/24  ether1          0
[admin@Router-SE] >
````

Podemos observar que hay 2 entradas. Estas corresponden a redes directamente conectadas que el propio router añade al configurarles una dirección IP en esas subredes (se pueden detectar por la letra D). Ahora vamos a añadir una entrada estática en la tabla de rutas. Para ello hacemos clic en + y debemos añadir:

* Subred de destino y máscara en formato CIDR.
* Puerta de enlace o próximo salto (router a atravesar).

Hacemos lo mismo para ambos routers:

* **Router-SE**:
![rutaestaticaSE](https://i.ibb.co/pfN44Wk/rutaestatica-SE.jpg)

Desde el terminal:
````terminal
[admin@Router-SE] > ip/route/add dst-address=192.168.41.0/24 gateway=1.1.1.41
````

* **Router-BACLN**:
![rutaestaticaBACLN](https://i.ibb.co/WGwWXcv/ritaestatica-BACLN.jpg)

Desde el terminal:
````terminal
[admin@Router-BACLN] > ip/route/add dst-address=192.168.21.0/24 gateway=1.1.1.21
````

Una vez añadida las dos entradas estáticas en ambos routers, podemos hacer una prueba de conectividad por ejemplo, realizando un ping desde **Cliente 1** a alguno de los equipos, accediendo a la página web o entrando al servidor por SSH:
