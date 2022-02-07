---
layout: post
title:  "Introducción a Cisco"
banner: "/assets/images/banners/Cisco-portada.png"
date:   2022-01-30 20:41:00 +0200
categories: Cisco
---

A partir de aquí empezaré una serie de post realizando configuraciones de switches y routers de Cisco. Mi idea es subir el contenido sin seguir un orden concreto, ya que no es mi intención hacer una especie de curso intensivo para preparar el CCNA ni nada de eso puesto que no soy CCNA (en estos momentos me encuentro preparándome para ello). Tampoco entraré en aspectos teóricos básicos a nivel de redes (salvo alguna excepción) por lo que mayoritariamente serán obviados. Lo que pretendo pues es subir distintos "laboratorios" que podeís encontrar en muchos sitios, aunque mi objetivo es explicarlos de tal manera que queden lo más claro posible según los conocimientos que yo he ido adquriendo.

Para los laboratorios utilizaré EVE-NG, ya que es el software de simulación de redes que más me gusta. Este software viene incluido en una máquina virtual, la cual se encargará de ejecutar a su vez todas las imágenes de los sistemas operativos de los dispositivos que iniciemos durante un laboratorio. Podeís descargar EVE-NG desde [aquí](https://www.eve-ng.net/). Para acceder se tiene que iniciar la máquina virtual. Esta contiene un sistema operativo GNU/LINUX (Ubuntu, en este caso). Como recomendación, deberiais darle un direccionamiento estático, ya que la manera de acceder a EVE-NG una vez encendida la máquina virtual es a través del navegador web escribriendo la dirección IP de la máquina virtual (por lo que no queremos que esta cambie en algún momento dado).

El procedimiento para configurar un direccionamiento estático a la máquina virtual lo explico a continuación:

Al iniciar la máquina virtual nos pide introducir un nombre de usuario y además, si nos fijamos, nos da la contraseña para el root y la dirección IP. El idioma del sistema operativo está en ingles, por lo que el teclado también estará en inglés, lo que significa que a la hora de ejecutar algunos comandos tendrémos que estar buscando cual es el caracter que queremos introducir. La forma más rápida de solucionar esto es accediendo a la máquina virtual un cliente SSH, ya que de esta manera si tendremos el idioma del teclado al español. Si tenéis Windows, podeis usar PUTTY. Para ello, copiamos la dirección IP que nos facilita la máquina virtual y accedemos vía SSH.

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

Ahora que hemos realizado los preparativos para posteriormente hacer los laboratorios, es hora de entrar en materia con Cisco. Antes de empezar a hablar de configuraciones, lo primero es hablar de lo que es el dispositivo físicamente para tener una idea de su funcionamiento real. 

La mayoría de dispositivos Cisco poseen tres tipos de puertos:

* Puerto de consola
* Puerto auxiliar
* Puertos Ethernet

Los puertos Ethernet son a los que conectamos los equipos de la red, el puerto de consola, al que nos conectamos con el llamado cable "Rollover" o cable de consola. Este último es el cable que se usa para realizar las primeras configuraciones en un dispositivo Cisco, ya sea un router o un switch, es decir, cuando el equipo no tiene ninguna configuración IP ni nada para poder conectarnos remotamente. El puerto auxiliar es para acceder a la CLI mediante un módem.

Supongamos que tenemos un router Cisco que no ha sido usado previamente, por lo que no tiene configuración alguna. Lo primero es conectarnos mediante un cable de consola al router. El cable de consola contiene en un extremo un conector RJ-45 y en el otro un conector puerto serie DB9 hembra. Los PC de hoy en día ya no traen consigo puerto serie, por lo que si no disponemos de un ordenador que tenga puerto serie, existen adaptadores de puerto serie a USB que podemos comprar a un precio razonable.

Una vez que tengamos conectado el cable de consola desde nuestro equipo al router accedemos a la CLI. En mi caso usaré PUTTY, donde seleccionaré el puerto COM (serie) adecuado y otras configuraciones. Yo estoy usando un adaptador de puerto serie a USB. Si usais un cable de este tipo debéis instalar el driver que os proporcione el fabricante.

Típicamente, los parámetros para la configuración del dispositivo 
mediante cable de consola son: 
* Puerto COM adecuado 
* 9600 baudios 
* 8 bits de datos 
* Sin paridad 
* 1 bit de parada 
* Sin control de flujo 

Desde PUTTY, esta configuración se puede realizar si nos vamos al menú de la izquierda, en el apartado "serial":
 
![putty](https://i.ibb.co/3r2pP0j/introduccion-a-cisco-putty.jpg "putty") 

En el apartado "Session" seleccionamos el puerto COM adecuado, el tipo de conexión "Serial" y la velocidad en 9600:

![putty](https://i.ibb.co/s1t4jpc/introduccion-a-cisco-putty2.jpg "putty")

Finalmente hacemos clic en "Open" y entraremos en el CLI de Cisco:

![putty](https://i.ibb.co/8rDsLWq/introduccion-a-cisco-putty3.jpg "putty")

Se puede apreciar en la imagen anterior que antes tuve que insertar una contraseña para poder entrar al CLI. Esto se hace estableciendo una contraseña para el acceso por consola, algo que veremos más adelante. La imagen mostrada corresponde a un router Cisco real que tengo por casa, ya que mediante EVE-NG no puedo hacer una demostración real para este caso.

Es importante conocer algunas características en cuanto al hardware de Cisco. Los switches y routers de Cisco contienen:

* Una memoria **flash**, que es el equivalente a un disco duro en un PC. En ella se almacena el software IOS de Cisco.
* Memoria **RAM** (Random Access Memory) en español, memoria de acceso aleatorio. En ella se guardan las configuraciones que están corriendo en un momento dado. Su contenido se pierde al apagar/reiniciar el dispositivo.
* **NVRAM** (Non-Volatile Random Access Memory) en español, memoria de acceso aleatorio no volátil. En ella se guardan las configuraciones de inicio y su contenido no se pierde al apagar/reiniciar el dispositivo.
* **ROM** (Read Only Memory) en español, memoria de solo lectura. Se utiliza para almacenar de forma permanente el código de diagnóstico de inicio (Monitor de ROM)

Durante el arranque del dispositivo, este lleva a cabo unas rutinas de detección del hardware. Para estas rutinas se emplea el término **POST** (Power-on Selft Test). Una vez que se ha comprobado todo el hardware y este se encuentra operativo, se procede al arranque del sistema IOS. Tras cargar el sistema operativo, se cargan las configuraciones almacenadas en memoria (si las hay).

A partir de ahora haremos algunas configuraciones básicas de inicio en nuestro dispositivo y veremos como poder guardar toda configuración introducida pero no sin antes hablar del CLI de Cisco.

Existen 3 modos principales en el CLI de Cisco:

* **Modo usuario**: es el primer modo de configuración. Cualquier usuario que tenga acceso al dispositivo pasará por este modo. Desde aquí no podemos hacer ninguna configuración al dispositivo ya que no tendremos los privilegios para ello. Lo único que podemos ejecutar son comandos "informativos". Cada modo nos muestra un símbolo cuya finalidad es informarnos de en qué modo estamos. El modo usuario muestra el símnomo "**>**". Para pasar al siguiente modo (privilegiado) tenemos que ejecutar el comando **enable**.
*  **Modo privilegiado**: dentro de este modo tendremos los privilegios para hacer configuraciones en el dispositivo. Además, desde aquí podemos acceder al modo que nos permite realizar la mayoría de las configuraciones (modo global) ejecutando el comando **configure terminal**. Para regresar al modo usuario se ejecuta **disable**. El modo privilegiado muestra el símbolo "**#**".
*  **Modo configuración global**: desde este modo podemos hacer la mayoría de las configuraciones como cambiar el nombre del dispositivo, configuración de IPs para las interfaces, enrutamiento, etc. El modo configuración global no muestra símbolo, en lugar de ello nos muestra entre paréntesis "(config)" para indicárnoslo. Para volver al modo privilegiado podemos ejecutar **exit**, **end** e incluso mediante la combinación de teclas "Control + z".

Como he dicho anteriormente, dependiendo del modo en que nos encontremos podemos ejecutar unos comandos u otros. Podemos ver qué comandos podemos ejecutar si introducimos el símbolo "**?**":

````console
Configure commands:
  aaa                         Authentication, Authorization and Accounting.
  access-list                 Add an access list entry
  alias                       Create command alias
  appfw                       Configure the Application Firewall policy
  archive                     Archive the configuration
  arp                         Set a static ARP entry
  async-bootp                 Modify system bootp parameters
  autoupgrade                 Auto Upgrade Manager simplifies image upgrade
                              process
  banner                      Define a login banner
  bba-group                   Configure BBA Group
  beep                        Configure BEEP (Blocks Extensible Exchange
                              Protocol)
  bfd                         BFD configuration commands
  boot                        Modify system boot parameters
  bridge                      Bridge Group.
  buffers                     Adjust system buffer pool parameters
  busy-message                Display message when connection to host fails
  call                        Configure Call parameters
  call-history-mib            Define call history mib parameters
  call-home                   Enter call-home configuration mode
  cdp                         Global CDP configuration subcommands
 --More--
````

Vamos ahora a realizar algunas configuraciones básicas: primero vamos a configurar un nombre al dispositivo. Esto es algo recomendado para así poder indentificar con mayor facilidad a los dispositivos en una red. Para ello debemos ir al modo de configuración global y ejecutamos: ````hostname nombredeldispositivo````.

````console 
Router#conf t 
Enter configuration commands, one per line. End with CNTL/Z. 
Router(config)#hostname ROUTER_SE 
ROUTER_SE(config)#
````

Se puede observar que a la hora de introducir ````configure terminal```` para acceder al modo de configuración global solo me ha bastado escribiendo el comando de una forma corta. El CLI de Cisco nos permite abreviar los comandos mientras existan coincidencias suficientes para saber qué comando es el que queremos ejecutar.

También es importante tener en cuenta dos detalles que nos ayudarán a poder buscar y ejecutar el comando que queremos: 
* Mediante la tecla "Tab" (tabulador), podemos hacer una predicción del comando (teniendo en cuenta lo de que 
tiene que coincidir lo suficiente con algún comando para que el CLI nos autocomplete el comando) 
* Si seguido del comando que queremos ejecutar introducimos el signo "**?**", nos cercioramos de que lo estamos escribiendo bien: 

````console 
ROUTER_SE(config)#hostname? 
hostname 
ROUTER_SE(config)#hostname
````
