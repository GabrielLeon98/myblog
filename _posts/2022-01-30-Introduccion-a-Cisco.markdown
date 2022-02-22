---
layout: post
title:  "Introducción a Cisco"
banner: "/assets/images/banners/Cisco-portada.png"
date:   2022-01-30 20:41:00 +0200
categories: Cisco
---

A partir de aquí empezaré una serie de post realizando configuraciones de switches y routers de Cisco. Mi idea es subir el contenido sin seguir un orden concreto, ya que no es mi intención hacer una especie de curso intensivo para preparar el CCNA ni nada de eso puesto que no soy CCNA (en estos momentos me encuentro preparándome para ello). Tampoco entraré en aspectos teóricos básicos a nivel de redes (salvo alguna excepción) por lo que mayoritariamente serán obviados. Lo que pretendo pues es subir distintos "laboratorios" que podeís encontrar en muchos sitios, explicándolos de manera breve y concisa en base a los conocimientos que he ido adquiriendo. Para los laboratorios, usaré el software simulador de redes **EVE-NG**, cuya instalación y manejo explico en el post [Configuración de EVE-NG](https://www.gabriellv.com/eve-ng/2022/02/12/Configuracion-EVE-NG.html).

Antes de empezar a hablar de configuraciones, creo que sería adecuado hablar de lo que es el dispositivo físicamente para tener una idea de su funcionamiento real. 

## El dispositivo físico. Primera conexión

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

Se puede apreciar en la imagen anterior que antes tuve que insertar una contraseña para poder entrar al CLI. Esto se hace estableciendo una contraseña para el acceso por consola, algo que veremos más adelante, pero lo normal es que al inciar por primera vez un router Cisco nos pida primero una serie de configuraciones mínimas a través de un diálogo opcional llamado **Setup**, como lo que aparece a continuación:

````console
   --- System Configuration Dialog ---

Would you like to enter the initial configuration dialog? [yes/no]:
````

La anterior imagen mostrada corresponde a un router Cisco real que tengo por casa, ya que mediante EVE-NG no puedo hacer una demostración real para este caso.

Es importante conocer algunas características en cuanto al hardware de Cisco. Los switches y routers de Cisco contienen:

* La **CPU**, que ejecuta las instrucciones del sistema operativo.
* Una memoria **flash**, que es el equivalente a un disco duro en un PC. En ella se almacena el software IOS de Cisco.
* Memoria **RAM** (Random Access Memory) en español, memoria de acceso aleatorio. En ella se guardan las configuraciones que están corriendo en un momento dado. Su contenido se pierde al apagar/reiniciar el dispositivo.
* **NVRAM** (Non-Volatile Random Access Memory) en español, memoria de acceso aleatorio no volátil. En ella se guardan las configuraciones de inicio y su contenido no se pierde al apagar/reiniciar el dispositivo.
* **ROM** (Read Only Memory) en español, memoria de solo lectura. Se utiliza para almacenar de forma permanente el código de diagnóstico de inicio (Monitor de ROM)

Durante el arranque del dispositivo, este lleva a cabo unas rutinas de detección del hardware. Para estas rutinas se emplea el término **POST** (Power-on Selft Test). Una vez que se ha comprobado todo el hardware y este se encuentra operativo, se procede al arranque del sistema IOS. Tras cargar el sistema operativo, se cargan las configuraciones almacenadas en memoria (si las hay).

A partir de ahora haremos algunas configuraciones básicas de inicio en nuestro dispositivo y veremos como poder guardar toda configuración introducida pero no sin antes hablar del CLI de Cisco.

## Primeros pasos en el CLI del IOS de Cisco

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
Para ver las configuraciones que vayamos añadiendo ejecutamos ````show running-config```` desde el modo privilegiado:

````console
Current configuration : 654 bytes
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname ROUTER_SE
!
boot-start-marker
boot-end-marker
````
**IMPORTANTE**: como hemos dicho más arriba, dependiendo del modo que nos encontremos en la CLI podremos ejecutar unos comandos u otros. Si estamos en el modo configuración global y queremos ejectuar un ````show running-config```` o cualquier otro comando que corresponda al modo privilegiado desde el modo configuración global debemos añadir ````do```` al principio de cada línea para de esta manera hacer una llamada al modo privilegiado. Con esto no tenemos que estar saliendo del modo configuración global cada vez que queramos ejecutar algún ````show```` u otro comando del modo privilegiado, lo que nos es bastante más cómodo.

Saber hacer uso de los comandos ````show```` es de gran importancia a la hora de hacer cualquier tipo de diágnostico de fallos. Desde el modo usuario se permite ejecutar los comandos ````show```` de manera restringida, pero desde el modo privilegiado, la cantidad es mayor. Algunos de los comandos ````show```` que podemos ejecutar (desde el modo privilegiado) son:

* **show interfaces**: muestra información detallada de las interfaces.
* **show protocols**: muestra información del protocolo de capa 3 configurado y el estado por interfaz.
* **show arp**: muestra la tabla ARP.
* **show running-config**: muestra el contenido del archivo de configuración que se está ejecutando.
* **show startup-config**: muestra el contenido del archivo almacenado en la NVRAM.
* **show history**: muesta el historial de los comandos ejecutados.
* **show flash**: muestra información sobre la memoria **flash** y los archivos almacenados.
* **show version**: muestra información acerca del dispositivo, de la imagen IOS y del **registro de configuración** (del que ya se hablará en otro post).

## Un poco de hardening

A continuación, vamos a restringir el acceso al modo privilegiado para dotar de cierta seguridad a nuestro dispositivo. Podemos hacerlo ejecutando dos comandos diferentes desde el modo de configuración global:

* ````enable password contraseña````: la contraseña no es cifrada.
* ````enable secret contraseña````: es la manera recomendada. La contraseña es cifrada utilizando el algoritmo MD5.

El hecho de que la contraseña sea cifrada es importante ya que si no lo hacemos, al ejecutar un ````show running-config```` o ````show startup-config```` en caso de tener la configuración almacenada, las contraseñas aparecerán en texto plano, por lo que si estamos manejando el dispositivo en presencia de alguien que no tiene permisos para acceder a estos, la persona podría ver la contraseña, lo cual pone en riesgo la seguridad de los equipos. Veamos la diferencia con un ejemplo real:

* Usando ````enable password````:

````console
Current configuration : 676 bytes
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname ROUTER_SE
!
boot-start-marker
boot-end-marker
!
!
enable password cisco
!
no aaa new-model

````
Si nos fijamos, la contraseña aparece en texto plano.

* Ahora vamos a hacer lo mismo pero con ````enable secret````:

````console
Current configuration : 714 bytes
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname ROUTER_SE
!
boot-start-marker
boot-end-marker
!
!
enable secret 4 tnhtc92DXBhelxjYk8LWJrPV36S2i4ntXrpb4RFmfqY
!

````

Como se puede apreciar, ahora la contraseña aparece cifrada.

Existe una utilidad que nos permite cifrar cualquier contraseña almacenada en el equipo de forma automática sin tener que recurrir al comando ````enable secret````. Para hacer uso de esta utilidad, desde el modo de configuración global debemos ejecutar el comando ````service password-encryption````, que encripta con un cifrado leve las contraseñas que no están cifradas por defecto (como las de telnet, consola, auxiliar, etc).

Hemos configurado una contraseña para el acceso al modo privilegiado, ahora vamos a ver como configurar una contraseña para el acceso mediante los puertos de consola, auxiliar y para el acceso remoto:

* Para el puerto de la consola, basta con ejecutar desde el modo de configuración global ````line console 0````. El 0 hace referencia a que solamente existe un puerto de consola en el dispositivo. Una vez dentro de la configuración del puerto, debemos asignar la contraseña ejecutando ````password contraseñaquequeramos````. Para que la contraseña configurada tenga efecto, ha de ejecutarse seguidamente el comando ````login```` con la finalidad de que al ingresar por el puerto de la consola, se nos pida introducir la contraseña configurada previamente:
````console
ROUTER_SE>en
Password:
ROUTER_SE#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ROUTER_SE(config)#line console 0
ROUTER_SE(config-line)#password 1234
ROUTER_SE(config-line)#login
ROUTER_SE(config-line)#
````

Con haber configurado una contraseña para el puerto de consola y otra para acceder al modo privilegiado, el dispositivo estará algo más seguro:

````console
User Access Verification

Password:
ROUTER_SE>enable
Password:
ROUTER_SE#
````

* Para el puerto auxiliar el procedimiento es el mismo, tenemos que ejecutar desde el modo de configuración global ````line aux 0````. Igualmente que con el puerto de consola, el número 0 hace referencia a que solo hay un puerto auxiliar. Introducimos la contraseña con ````password contraseñaquequeramos```` y de nuevo ````login````.
* Para el acceso remoto se han de configurar los puertos virtuales (**vty**), que son puertos utilizados para conexiones remotas a través de telnet o SSH. Para ello desde el modo de configuración global ejecutamos ````line vty 0 4````. En este caso, el 0 hace referencia al número de la interfaz y el 4 al número máximo de conexiones múltiples a partir de 0. En este caso se permiten 5 conexiones múltiples. Tras ello, ejecutamos ````password contraseñaquequeramos```` y por último ````login```` para que nos pida la contraseña al conectarnos. Para probar el funcionamiento en este caso, se ha de tener configurada una dirección IP en alguna interfaz del dispositivo para probar la conexión remota, algo de lo que hablaremos a continuación. En otro post, hablaré también de cómo configurar SSH en un dispositivo Cisco.
````console
ROUTER_SE#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ROUTER_SE(config)#line vty 0 4
ROUTER_SE(config-line)#password cisco
ROUTER_SE(config-line)#
````

También existe la posibilidad de dar un añadido de seguridad creando usuarios con sus respectivas contraseñas con los que poder autenticarse a la hora de acceder a la configuración de los dispositivos. Para crear un usuario basta con ejecutar desde el modo de configuración global ````username usuario password contraseña````. De esta manera la contraseña aparecerá en texto plano. Existe otra posibilidad que es ejecutar ````username usuario secret contraseña```` y utilizará un algortimo MD5 para el cifrado de la contraseña:
````console
username pepe secret 4 tnhtc92DXBhelxjYk8LWJrPV36S2i4ntXrpb4RFmfqY
````

Si queremos que se nos pida autenticarnos a la hora de acceder al dispositivo por ssh o mediante el cable de consola, debemos ejecutar desde el modo de configuración de línea ````login local```` para que se habilite la base de datos local para la autenticación:
````console
User Access Verification

Username: pepe
Password:
Switch>
````

## Asignación de direcciones IP

Vamos a ver ahora como podemos asignar direcciones IP a las distintas interfaces del dispositivo, aun que se han de tener en cuenta algunas diferencias a la hora de asignar direcciones IP a un router o a un switch.

### Asignación de direcciones IP a un switch de Cisco.

Para configurar una dirección IP en un switch de ha de hacer sobre una interfaz **vlan**. No quiero entrar demasiado en detalle con las vlan (al menos ahora), pero en todos los switches viene por defecto una vlan que es denominada la *vlan nativa* que corresponde a la vlan con el **ID** número 1. Esta vlan es considerada como la vlan de *administración*, al configurar una dirección IP a la *vlan 1* podremos administrar el switch remotamente por telnet o SSH. Para ello desde el modo de configuración global ejecutamos ````interface vlan 1```` para entrar al **modo de configuración de interfaz**, posteriormente, ejecutamos ````ip address IP Máscara`````quedando de tal manera:
````console
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface vlan 1
Switch(config-if)#ip address 192.168.0.20 255.255.255.0
Switch(config-if)#
````

Este método de direccionamiento es estático, pero tenemos la posibilidad de configurar un direccionamiento dinámico por medio de un servidor DHCP ejecutando ````ip address dhcp````. Con un servidor DHCP obtenemos los parámetros de red necesarios, pero de manera estática solamente proporcionamos un direccionamiento lógico al dispositivo, por lo que si queremos que el switch envíe paquetes a una red diferente a la de administración, debemos configurar un gateway ejecutando desde el modo de configuración global ````ip default-gateway direccionIP````.

Un detalle a tener en cuenta a la hora de configurar las interfaces o puertos de un dispositivo Cisco es que estas vienen en off, es decir, no están activas, por lo que a parte de configurarlas, debemos activarlas/encenderlas. Para ello tras la configuración debe ejecutarse desde el modo de configuración de interfaz ````no shutdown````:
````console
Switch(config)#interface vlan 1
Switch(config-if)#no shutdown
Switch(config-if)#
*Feb 21 16:10:31.038: %LINK-3-UPDOWN: Interface Vlan1, changed state to up
*Feb 21 16:10:32.050: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
Switch(config-if)#
````

Para volver a desactivar un puerto, ejecutamos ````shutdown````.

### Asignación de direcciones IP a un router de Cisco

Todo lo anterior es aplicable a un router de Cisco, solo que en este caso configuraremos interfaces Ethernet (o serial):
````console
ROUTER_SE#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ROUTER_SE(config)#interface fas
ROUTER_SE(config)#interface fastEthernet 0/0
ROUTER_SE(config-if)#ip address 192.168.0.30 255.255.255.0
ROUTER_SE(config-if)#no sh
ROUTER_SE(config-if)#no shutdown
ROUTER_SE(config-if)#
*Feb 21 19:08:46.851: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Feb 21 19:08:47.851: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
ROUTER_SE(config-if)#
````

Los dispositivos Cisco llevan ranuras o slots donde se instalan los módulos de interfaces o para ampliar la cantidad de estos. Los slots y las interfaces son numerados de la siguiente manera:
````console
Router(config)# interface tipo slot/interfaz
````

Otros parámetros básicos de configuración para las interfaces son:

* La velocidad de transmisión: ````speed [10|100|1000|auto].
* El duplex: ````duplex [auto|full|half].

Además, desde un router podemos crear interfaces lógicas llamadas subinterfaces que pueden ser usadas como interfaces independientes pero que se encuentran dentro de una interfaz física. Para ello desde el modo de configuración global ejecutamos ````interface número.número de interfaz````:
````console
ROUTER_SE(config)#interface f1/0
ROUTER_SE(config-if)#no sh
ROUTER_SE(config-if)#no shutdown
ROUTER_SE(config-if)#
*Feb 22 08:17:00.115: %LINK-3-UPDOWN: Interface FastEthernet1/0, changed state to up
*Feb 22 08:17:01.115: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0, changed state to up
ROUTER_SE(config-if)#interface f1/0.1
ROUTER_SE(config-subif)#
````

Es importante activar la interfaz física para que las interfaces lógica funcionen, ya que estas últimas dependen de la primera. Este tipo de interfaces son empleadas por ejemplo para realizar enrutamiento entre VLANs (ROAS), algo que ya veremos en otra ocasión.

Tras realizar cualquier direccionamiento es posible verificarlo ejecutando el comando ````show ip interface brief```` desde el modo privilegiado (o desde el modo global añadiendo do por delante):
````console
ROUTER_SE(config)#do show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        192.168.0.30    YES NVRAM  up                    up
FastEthernet1/0        unassigned      YES NVRAM  administratively down down
FastEthernet2/0        unassigned      YES NVRAM  administratively down down
ROUTER_SE(config)#
````
