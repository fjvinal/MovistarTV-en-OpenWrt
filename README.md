# Configuración de la IPTV de Movistar Plus con OpenWrt
## Hardware usado:
* ONT; Huawei EchoLife HG8010H
* Router: GL-iNet Convexa-B (GL-B1300). Cuenta con un puerto WAN y dos puertos LAN
* Sustituyen al router proporcionado por Telefónica: Askey RTF3505VW (HGU)
## Versión de OpenWrt usada:
* OpenWrt 23.05.0-rc3
## Instrucciones:
Para la mayor parte de las modificaciones se puede usar la interfaz web Luci (http://192.168.1.1/cgi-bin/luci/), la principal excepción es la configuración de igmpproxy.
Pongo directamente las modificaciones en los archivos de '/etc/config' para que la exlicación sea más breve.
Para ello hay que entrar al router por ssh (ssh root@192.168.1.1) y usar un editor como 'nano' o el propio 'vi' que viene preinstaldo si se prefiere. 
### Configuración básica del router:
* Cambiar la contraseña
* Cambiar la zona horaria: /etc/config/system
* Limitar el acceso por ssh a la zona LAN: etc/config/dropbear
### Instalación de los paquetes necesarios:
* opkg update
* opkg install nano igmpproxy kmod-nft-bridge kmod-ipt-nathelper-rtsp
### Configuración de la red: /etc/config/network
* En el 'device br-lan' se activa el 'igmp-snooping' con las mismas opciones que el router Askey
* En el 'interface lan' borro las opciones relacionadas con IPv6 ya que Telefónica no proporciona este servicio
* Creo la 'interface wan.6" para la VLAN de acceso a Internet con MTU 1492
* Configuro la 'interface wan' con protocolo PPPoE que es el que usa Telefónica 
* Creo la 'interface wan.2' para la IPTV
* Configuro la 'interface iptv' con ip fija que hay que obtener de la configuración del router Askey, el resto de los valores es probable que no cambien
* La 'interface wan6' se puede dejar como está o borrarla
* Configuro cuatro rutas estáticas para 'interface iptv' copiadas del router Askey, otra opción es instalar el paquete 'bird2' para configurarlas automáticamente, pero exige permisos más amplios para Telefónica en el archivo 'firewall'
### Configuración del cortafuegos: /etc/config/firewall
* Dejo como está todo y añado lo que sigue al final
* Configuro 'zone iptv', en caso de usar 'bird2' para configurar las rutas hay que poner: option input 'ACCEPT'
* Configuro 'forwading lan iptv'
* Configuro dos reglas adicionales tipo 'input', lo debería hacer el script de igmpproxy, pero parece que está incompleto
### Configuración de DHCP y DNS: /etc/config/dhcp
* En 'dhcp lan' cambio el límite a 100
* Añado la etiqueta 'tag decotv' con las opciones para DHCP del decodificador ARRIS copiadas de las del router Askey
* Creo un 'host' para el decodificador ARRIS con IP fija 192.168.1.200, para saber la MAC del deco hay que encenderlo y verla en la página principal de Luci o ejecutando la orden 'arp' en el shell de nuestra sesión ssh
* Se pueden tener otros dos decodificadores usando las direcciones siguientes 101 y 102, aunque esto no lo he probado
### Configuración del proxy del protocolo IGMP: /etc/config/igmpproxy
* Se sustituye el ejemplo por el archivo que figura aquí
