# Lab-Monitoreo-de-Routers-Mikrotik-Cisco-y-Linux-Debian-9
## Laboratorio hecho en GNS3 con Mikrotik, Cisco, Ubuntu y Debian 

La topología de este laboratorio de monitoreo está compuesta de la siguiente forma
- Un servidor con Ubuntu 18.04 dónde está Instalado Zabbix 5.0, que será nuestro servidor de monitoreo. Este Router se monitorea con el protocolo SNMP
- Un Router Mikrotik con las redes 10.0.0.0/24 y 10.0.2.0/24, en la interface ethernet2 tiene un VPC con la IP 10.0.2.253. Este Router se moniterea con el protocolo SNMP
- Un Router Cisco que forma parte de la red 10.0.0.0/24 y tiene un Debian conectado en su interface FastEthernet1/0, con la ip 10.0.1.3/24. Esta máquina Debian se monitorea con un agente de Zabbix.

## Topologia del laboratorio:

![Alt text](https://imgur.com/2e2ZpRz.png)

## Instalación de Zabbix 5.0 en Ubuntu 18.04

Zabbix es una herramienta de software de monitoreo de código abierto para diversos componentes de TI, incluidas redes, servidores, máquinas virtuales (VM) y servicios en la nube. Zabbix proporciona métricas de monitoreo, como la utilización de la red, la carga de la CPU y el consumo de espacio en disco. El software supervisa las operaciones en Linux y otras plataformas.

## Supervise las posibles métricas de rendimiento y los incidentes en la red:

### Rendimiento de la red:
 
- Uso del ancho de banda de la red
- Tasa de pérdida de paquetes
- Tasa de error de interfaz
- Alta utilización de CPU o memoria
- El número de conexiones tcp es anormalmente alto para este día de la semana

### Salud de la red:
 
- El enlace está caído
- El estado del sistema es de advertencia/estado crítico
- La temperatura del dispositivo es demasiado alta/demasiado baja
- La fuente de alimentación está en estado crítico
- El espacio libre del disco es bajo
- Sin recopilación de datos SNMP
 

### Cambios de configuración:
 
- Nuevo dispositivo agregado o eliminado
- Se agrega, quita o reemplaza el módulo de red
- Se actualizó el firmware
- El número de serie del dispositivo ha cambiado
- La interfaz ha cambiado a modo de velocidad más baja o semidúplex

Esta es una lista de muestra de métricas e incidentes relacionados con la red, monitoreados por Zabbix desde el primer momento.

Para la instalación de Zabbix, se seguirá la documentación oficial de Zabbix. Solo hay que tener instalado previamente un servidor web, en este caso Apache2 y un motor de base de datos, en este caso usaré Mysql.

### Verificacion de que Mysql y Apache2 estén instalados:
`sudo systemctl status mysql`

`sudo systemctl status apache2`

Los pasos a seguir son los de la documentacion oficial para apache2 y mysql, https://www.zabbix.com/la/download?zabbix=5.4&os_distribution=ubuntu&os_version=18.04_bionic&db=mysql&ws=apache

Una vez que esté instalado Zabbix, accedemos a su frontend de esta forma
http://localhost/zabbix
En este punto hay que seguir los pasos que se indican, una vez que esté terminado se ingresa el usuario `Admin` y la contraseña por defecto
`zabbix`.

![Alt text](https://imgur.com/mbzD7sR.png)

## Verificación de conectividad entre los dispositivos de red y el servidor Zabbix

Antes de la configuracion del monitoreo de los dispositivos, verificaré que los routers puedan hacer ping correctamente al servidor. La IP del servidor Zabbix es la 10.0.0.253

**Ping desde el Mikrotik hacia el Servidor Zabbix:** 

![Alt text](https://imgur.com/shRIDlK.png)

**Ping desde el Cisco hacia el Servidor Zabbix:**

![Alt text](https://imgur.com/vaMCxQW.png)


## SNMP y su configuracion en dispositivos de red

### Soporte SNMP
Un servidor Zabbix puede recopilar datos de dispositivos con versiones de agente SNMP v1, v2 o v3. Los agentes SNMP están presentes no solo en equipos de red, sino también en impresoras, NAS, UPS. Básicamente, cualquier equipo decente que esté presente en la red se puede monitorear a través de agentes SNMP. Las comprobaciones SNMP se realizan únicamente a través del protocolo UDP.

## Configuración de SNMP en un Router Cisco.

Con el comando `configure terminal` se ingresa al modo de configuración.

`R1-CISCO# configure terminal`

Con los siguientes comandos se habilita y configura el servicio Cisco SNMP.

~~~
R1-CISCO(config)# snmp-server community public ro
R1-CISCO(config)# snmp-server contact Diego <diegoadm@prueba.com>
R1-CISCO(config)# snmp-server location Planta baja - Piso IT
R1-CISCO(config)# exit
~~~

- La Comunidad public tiene permiso de solo lectura en el Router Cisco.
- La persona de contacto responsable de este Router se configuró como Diego.
- La ubicación del equipo se configuró como la Sala de TI del Universo 10.


Para probar la configuración SNMP, use los siguientes comandos en el equipo con Ubuntu Linux Zabbix Server![snmp mikro](https://user-images.githubusercontent.com/88456338/128647700-682c1124-5156-411b-be52-08eef878e69b.png)
.

`# snmpwalk -v2c -c public 10.0.0.254`

Pequeña muestra de la salida de SNMPWALK:
![capture-20210808-192434](https://user-images.githubusercontent.com/88456338/128647439-df1bfd9d-0d56-43c8-8e10-df2970adabdf.png)

## Configuración de SNMP en un Router Mikrotik.

Para habilitar el servicio SNMP hay que seleccionar la ventana IP y luego en SNMP.
- Se configura la comunidad public
- La persona de contacto responsable del Router es Juan
- La ubicacion se establece en Segundo Piso

![snmp](https://user-images.githubusercontent.com/88456338/128647641-cec1c17d-11a0-4638-aefd-2a59e92a7721.png)

Para probar la configuración SNMP, use los siguientes comandos en el equipo con Ubuntu Linux Zabbix Server.

`# snmpwalk -v2c -c public 10.0.0.1`

Pequeña muestra de la salida de SNMPWALK:
![snmp mikro](https://user-images.githubusercontent.com/88456338/128647705-cbb476e1-544a-43dd-8f00-524a4c10731c.png)


