# Informe de Desarrollo - Práctica 2: Red de la Ciudad Comercial Cayalá

**Curso:** Redes de Computadoras 1 - 2S2026
**Carnet:** 201708993
**Herramienta:** Cisco Packet Tracer (switches 3560 y 2960, solo CLI)

## 1. Resumen

Se implementó una red de cinco zonas de Cayalá segmentada con VLANs (13, 23, 33, 43 y 53), interconectada con una topología híbrida de estrella, malla parcial y anillo. Cada zona tiene un switch principal 3560 y un switch de acceso 2960. Se configuraron VTP (dominio 201708993), Rapid PVST+ con root bridge explícito por VLAN, y un EtherChannel PAgP (el último dígito del carnet es 3, impar) entre las Zonas 1 y 2. El detalle del diseño, el direccionamiento y la justificación de cada enlace están en `manual_tecnico.md`.

## 2. Procedimiento de implementación

### 2.1 Cableado y nombres

Se colocaron 5 switches 3560-24PS (SW-Z1-P a SW-Z5-P), 5 switches 2960-24TT (SW-Z1-A a SW-Z5-A) y 15 PCs (3 por zona). Se usó cable cruzado entre switches y directo hacia los PCs. El cableado se verificó con `show cdp neighbors`.

![Topología completa](capturas/01_topologia_completa.png)

![Vecinos CDP de SW-Z2-P](capturas/02_cdp_vecinos_SW-Z2-P.png)

### 2.2 Configuración base y VTP

Se aplicó la primera parte de cada script: nombre del dispositivo, VTP (servidores en los cinco switches principales, clientes en los de acceso), Rapid PVST+, troncales con VLAN nativa 99 y VLANs permitidas restringidas, y EtherChannel. Las VLANs 13, 23, 33, 43, 53, 99 y 999 se crearon únicamente en SW-Z2-P y los demás switches las recibieron por VTP, con la misma revisión de configuración.

![VLANs en SW-Z2-P](capturas/03_vlan_brief_SW-Z2-P.png)

![VTP en el servidor](capturas/04_vtp_status_SW-Z2-P.png)

![VTP en un cliente](capturas/05_vtp_status_SW-Z1-A.png)

### 2.3 Troncales

Todos los enlaces entre switches son troncales explícitos con VLAN nativa 99 y solo las VLANs necesarias en cada enlace, sin usar `allow all`.

![Troncales de SW-Z2-P](capturas/11_trunk_SW-Z2-P.png)

![Troncal de SW-Z1-A](capturas/12_trunk_SW-Z1-A.png)

### 2.4 STP, puertos de acceso y Blackhole

Cuando las VLANs ya estaban sincronizadas se aplicó la segunda parte: prioridades de Rapid PVST+, interfaces VLAN (SVI) en SW-Z2-P, puertos de acceso de los hosts (Fa0/2-4 en la VLAN de su zona, con PortFast y BPDU Guard) y envío de todos los puertos sin uso a la VLAN Blackhole 999 en estado `shutdown`.

![SW-Z1-A](capturas/06_vlan_brief_SW-Z1-A.png)

![SW-Z2-A](capturas/07_vlan_brief_SW-Z2-A.png)

![SW-Z3-A](capturas/08_vlan_brief_SW-Z3-A.png)

![SW-Z4-A](capturas/09_vlan_brief_SW-Z4-A.png)

![SW-Z5-A](capturas/10_vlan_brief_SW-Z5-A.png)

Root bridge por VLAN (prioridad 4096 en el switch principal de cada zona):

![VLAN 13](capturas/13_stp_vlan13_SW-Z1-P.png)

![VLAN 23](capturas/14_stp_vlan23_SW-Z2-P.png)

![VLAN 33](capturas/15_stp_vlan33_SW-Z3-P.png)

![VLAN 43](capturas/16_stp_vlan43_SW-Z4-P.png)

![VLAN 53](capturas/17_stp_vlan53_SW-Z5-P.png)

![VLAN 99](capturas/18_stp_vlan99_SW-Z2-P.png)

### 2.5 EtherChannel

Entre SW-Z2-P y SW-Z1-P se agruparon Gi0/1 y Gi0/2 con PAgP en modo `desirable`. La Zona 1 es la de mayor volumen de hosts (60) y de tráfico comercial, por lo que se le dio doble capacidad de enlace y tolerancia a la caída de uno de los cables.

![EtherChannel en SW-Z2-P](capturas/19_etherchannel_SW-Z2-P.png)

![EtherChannel en SW-Z1-P](capturas/20_etherchannel_SW-Z1-P.png)

### 2.6 Direccionamiento y enrutamiento

A cada PC se le asignó IP estática, máscara y default gateway según la tabla del manual. El switch multicapa SW-Z2-P tiene una SVI por zona (`.1` de cada red) con `ip routing`, lo que permite la comunicación entre todas las zonas.

![Rutas en SW-Z2-P](capturas/21_ip_route_SW-Z2-P.png)

![Interfaces en SW-Z2-P](capturas/22_ip_interface_brief_SW-Z2-P.png)

## 3. Pruebas de conectividad

### 3.1 Dentro de cada zona (10 pings)

Desde un host de cada zona se ejecutó `ping -n 10` hacia otro host de la misma zona, con 10 respuestas exitosas.

![Zona 1](capturas/23_ping_zona1.png)

![Zona 2](capturas/24_ping_zona2.png)

![Zona 3](capturas/25_ping_zona3.png)

![Zona 4](capturas/26_ping_zona4.png)

![Zona 5](capturas/27_ping_zona5.png)

### 3.2 Entre zonas

Desde PC-Z1-1 se probó la conectividad hacia hosts de las Zonas 2, 3, 4 y 5.

![Zona 1 a Zona 2](capturas/28_ping_z1_a_z2.png)

![Zona 1 a Zona 3](capturas/29_ping_z1_a_z3.png)

![Zona 1 a Zona 4](capturas/30_ping_z1_a_z4.png)

![Zona 1 a Zona 5](capturas/31_ping_z1_a_z5.png)

### 3.3 Redundancia

Se apagó la interfaz Gi0/1 de SW-Z2-P (`shutdown`). El Port-channel 1 siguió activo con el enlace restante y la conectividad hacia la Zona 1 se mantuvo. Después se reactivó la interfaz con `no shutdown`.

![Enlace caído](capturas/32_redundancia_enlace_caido.png)

![Ping con un enlace caído](capturas/33_redundancia_ping.png)

### 3.4 Análisis de ARP e ICMP

Al probar con paquetes simples en Packet Tracer, el primer paquete hacia un destino nuevo falló y los siguientes funcionaron. El motivo es ARP: antes de enviar el ICMP, el host debe resolver la MAC de su gateway (la SVI de SW-Z2-P) mediante un broadcast. El switch multicapa, a su vez, resuelve por ARP la MAC del host destino en la otra VLAN. Mientras esas tablas se llenan, el primer paquete se pierde. Una vez en caché, el flujo es: ICMP del host al gateway, enrutamiento entre SVIs en SW-Z2-P y entrega al host destino dentro de su VLAN.

![Simulación ARP e ICMP](capturas/34_simulacion_arp_icmp.png)

## 4. Problemas encontrados y soluciones

| # | Problema | Causa | Solución |
|---|---|---|---|
| 1 | Los comandos de configuración daban `Invalid input` y `no ip domain-lookup` quedaba colgado intentando resolver un nombre por DNS | Se pegó el script en modo usuario (`Switch>`) sin `enable`, y el primer comando se perdió porque el switch seguía arrancando | Esperar al prompt, escribir `enable` y pegar los scripts en bloques de 15 a 20 líneas, revisando que no aparezcan mensajes de error |
| 2 | Cuatro de los cinco switches de acceso estaban conectados al switch principal de otra zona | Conexión automática de cables en Packet Tracer, que usa el primer puerto libre. Se detectó por avisos `NATIVE_VLAN_MISMATCH` de CDP y al revisar la topología | Reconectar los cables eligiendo el puerto manualmente y verificar con `show cdp neighbors` en cada switch principal |
| 3 | Se aplicó la configuración de otro dispositivo a un switch por confundir el número | Error al elegir el switch | Resetear el equipo con `write erase`, `delete flash:vlan.dat` y `reload`. Se borró el `vlan.dat` porque guarda las VLANs y el número de revisión de VTP, y un switch con revisión alta podría sobrescribir las VLANs del resto |
| 4 | Asignar los puertos de acceso en los clientes VTP antes de que las VLANs sincronizaran | Un cliente solo puede usar VLANs que ya recibió del servidor | Dividir cada script en dos partes: base y VTP primero, y STP, SVIs, puertos de acceso y Blackhole después de comprobar con `show vlan brief` que las VLANs sincronizaron |
| 5 | El EtherChannel quedaba suspendido: `Po1(SD)` con `Gig0/1(s)` y `Gig0/2(s)` y el mensaje `%EC-5-CANNOT_BUNDLE2: Gig0/x is not compatible with Po1 and will be suspended (vlan mask is different)` | Primero, un lado tenía la VLAN nativa 99 y el otro la 1 por no haber aplicado aún la configuración. Después, al configurar el trunk en los puertos físicos antes de crear el `channel-group`, el Port-channel se creaba con valores por defecto y no coincidía con sus miembros | Quitar el `channel-group`, eliminar el Port-channel y recrearlo en el orden correcto: miembros solo con `channel-protocol` y `channel-group`, y el trunk (encapsulación, nativa 99 y VLANs permitidas) en la interfaz `port-channel`. Resultado: `Po1(SU)` con ambos puertos en `(P)` |
| 6 | `show running-config interface ...` daba `Invalid input` | Ese formato no está soportado por el IOS de Packet Tracer | Usar `show interfaces <interfaz> switchport` para revisar la configuración troncal |
| 7 | Los primeros pings entre zonas fallaban | Resolución ARP del primer paquete hacia un destino nuevo (ver 3.4) | Repetir la prueba. Con las tablas ARP llenas, todos los pings entre zonas fueron exitosos |

Evidencias de los problemas (opcionales):

![Cableado incorrecto](capturas/35_problema_cableado.png)

![EtherChannel suspendido](capturas/36_problema_etherchannel_suspendido.png)

![Comandos en modo usuario](capturas/37_problema_modo_usuario.png)

## 5. Conclusiones

- La segmentación por VLANs con troncales restringidos permitió aislar cada zona en capa 2 y comunicarlas solo a través del switch multicapa.
- Separar la configuración en fases (base y VTP, luego el resto) evitó errores con las VLANs que aún no existían en los clientes.
- En EtherChannel el orden de configuración es tan importante como los parámetros: el Port-channel debe tener el trunk configurado y coincidir con los miembros.
- Verificar el cableado con CDP antes de configurar ahorra tiempo de diagnóstico.

## 6. Entregables del repositorio

- `cayala.pkt`: simulación en Packet Tracer
- `scripts/`: comandos CLI de los 10 switches
- `manual_tecnico.md`: diseño, tablas y justificaciones
- `informe_desarrollo.md`: este documento
- `capturas/`: evidencias
