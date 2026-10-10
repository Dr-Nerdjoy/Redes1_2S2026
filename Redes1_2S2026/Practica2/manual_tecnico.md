# Manual Técnico - Práctica 2: Red de la Ciudad Comercial Cayalá

**Curso:** Redes de Computadoras 1 - 2S2026
**Carnet:** 201708993 (último dígito X = 3)
**Software:** Cisco Packet Tracer - switches Cisco 3560 y 2960 configurados solo por CLI

## 1. Datos derivados del carnet

| Dato | Valor |
|---|---|
| Último dígito (X) | 3 |
| VLANs de zona | 13, 23, 33, 43, 53 |
| Protocolo EtherChannel | **PAgP** (dígito impar) |
| Dominio VTP | 201708993 |
| Contraseña VTP | Cayala2026 |
| VLAN nativa | 99 |
| VLAN Blackhole | 999 |

## 2. Análisis de zonas

Las zonas se definieron a partir de fuentes públicas sobre Ciudad Cayalá, un desarrollo de usos mixtos con comercio, restaurantes y cine en Paseo Cayalá, oficinas, clínicas, laboratorios, bancos, gimnasios, escuelas y academias, además de vivienda (ver referencias al final de esta sección).

| Zona | Establecimientos | Rol operativo | Hosts | Tráfico | Criticidad | Enlaces |
|---|---|---|---|---|---|---|
| 1 | Paseo Cayalá (tiendas) | Corazón comercial; visitantes y comercios | 60 | Alto | Alta | 3 (Po1 de 2 enlaces + 1 a Z4) |
| 2 | Oficinas y administración central | Administración de la red, oficinas y seguridad; aloja el enrutamiento | 28 | Medio | Crítica | 5 (núcleo) |
| 3 | Clínicas, laboratorios y bancos | Servicios médicos y bancarios | 12 | Bajo/Medio | Media | 2 (Z2 y Z4) |
| 4 | Restaurantes, cafés y cine | Gastronomía y entretenimiento | 50 | Alto | Alta | 3 (Z2, Z1, Z3) |
| 5 | Gimnasio, academias y servicios complementarios | Servicios secundarios | 7 | Bajo | Baja | 1 (Z2) |

**Referencias de la investigación**

- Sitio oficial: https://cayala.com/
- ICSC, "La otra mitad de Ciudad de Guatemala": https://www.icsc.com/news-and-views/sct-iberoamerica/la-otra-mitad-de-ciudad-de-guatemala
- Living in Guatemala, "Cayalá - Datos rápidos": https://livinginguatemala.com/es/cayala/
- Wikipedia, Ciudad Cayalá: https://es.wikipedia.org/wiki/Ciudad_Cayal%C3%A1_%28Guatemala%29

## 3. Topología híbrida

Combina **tres** tipos de topología:

- **Estrella:** SW-Z2-P es el núcleo y conecta a Z1, Z3, Z4 y Z5, además de su switch de acceso.
- **Malla parcial:** Z1-Z4 tienen enlace directo y además ambas llegan a Z2.
- **Anillo:** Z2-Z3-Z4 forman un anillo (Z2-Z3, Z3-Z4, Z4-Z2).
- **Dentro de cada zona:** estrella (principal 3560 + acceso 2960).

### 3.1 Enlaces físicos y justificación

| # | Origen | Destino | Tipo | VLANs permitidas | Justificación |
|---|---|---|---|---|---|
| 1-2 | SW-Z2-P Gi0/1-2 | SW-Z1-P Gi0/1-2 | EtherChannel PAgP (Po1), crossover, Gigabit | 13,43,99 | Z1 tiene 60 hosts y tráfico alto: se agrega capacidad (2 Gbps) y redundancia |
| 3 | SW-Z2-P Fa0/1 | SW-Z4-P Fa0/1 | Troncal, crossover | 13,33,43,99 | Segundo camino de Z4 y camino de respaldo para Z1 y Z3 |
| 4 | SW-Z1-P Fa0/1 | SW-Z4-P Fa0/2 | Troncal, crossover | 13,43,99 | Malla parcial entre las dos zonas de mayor tráfico |
| 5 | SW-Z2-P Fa0/2 | SW-Z3-P Fa0/1 | Troncal, crossover | 33,99 | Enlace principal de Z3 |
| 6 | SW-Z3-P Fa0/2 | SW-Z4-P Fa0/3 | Troncal, crossover | 33,99 | Cierra el anillo; segundo enlace de Z3 |
| 7 | SW-Z2-P Fa0/3 | SW-Z5-P Fa0/1 | Troncal, crossover | 53,99 | Zona de baja criticidad: un solo enlace |
| 8-12 | SW-Zx-P | SW-Zx-A (Fa0/1) | Troncal, crossover | VLAN de zona + 99 | Distribución dentro de cada zona |
| - | SW-Zx-A Fa0/2-4 | PCs | Acceso, straight-through | VLAN de zona | Hosts finales |

### 3.2 Medios y cableado (TIA/EIA-568B)

- Switch-switch: **crossover**. Switch-PC: **straight-through**.
- Medios distintos: FastEthernet 100BASE-TX en acceso y distribución, y GigabitEthernet 1000BASE-T en el EtherChannel.
- Los modelos 3560-24PS y 2960-24TT de Packet Tracer solo ofrecen puertos de cobre, por lo que el uso distinto de medios se resolvió combinando FastEthernet (100BASE-TX) en acceso y distribución con GigabitEthernet (1000BASE-T) en el EtherChannel.

## 4. Segmentación y direccionamiento

| Zona | VLAN | Nombre | Red | Máscara | Gateway (SVI en SW-Z2-P) | Rango de hosts | Hosts útiles |
|---|---|---|---|---|---|---|---|
| 1 | 13 | Z1_PASEO_COMERCIAL | 192.168.13.0 | /26 | 192.168.13.1 | .2 - .62 | 62 |
| 2 | 23 | Z2_ADMIN_OFICINAS | 192.168.23.0 | /27 | 192.168.23.1 | .2 - .30 | 30 |
| 3 | 33 | Z3_MEDICOS_BANCOS | 192.168.33.0 | /28 | 192.168.33.1 | .2 - .14 | 14 |
| 4 | 43 | Z4_GASTRO_CINES | 192.168.43.0 | /26 | 192.168.43.1 | .2 - .62 | 62 |
| 5 | 53 | Z5_SERV_SECUNDARIOS | 192.168.53.0 | /28 | 192.168.53.1 | .2 - .14 | 14 |
| - | 99 | NATIVA_ADMIN | - | - | - | - | - |
| - | 999 | BLACKHOLE | - | - | - | - | - |

### Hosts de prueba (3 por zona)

| Zona | Host | IP | Máscara | Gateway | Switch / Puerto |
|---|---|---|---|---|---|
| 1 | PC-Z1-1 / 2 / 3 | 192.168.13.2 / .3 / .4 | 255.255.255.192 | 192.168.13.1 | SW-Z1-A Fa0/2-4 |
| 2 | PC-Z2-1 / 2 / 3 | 192.168.23.2 / .3 / .4 | 255.255.255.224 | 192.168.23.1 | SW-Z2-A Fa0/2-4 |
| 3 | PC-Z3-1 / 2 / 3 | 192.168.33.2 / .3 / .4 | 255.255.255.240 | 192.168.33.1 | SW-Z3-A Fa0/2-4 |
| 4 | PC-Z4-1 / 2 / 3 | 192.168.43.2 / .3 / .4 | 255.255.255.192 | 192.168.43.1 | SW-Z4-A Fa0/2-4 |
| 5 | PC-Z5-1 / 2 / 3 | 192.168.53.2 / .3 / .4 | 255.255.255.240 | 192.168.53.1 | SW-Z5-A Fa0/2-4 |

## 5. VTP

- Servidores: SW-Z1-P, SW-Z2-P, SW-Z3-P, SW-Z4-P, SW-Z5-P. Clientes: los cinco SW-Zx-A.
- Dominio `201708993`, contraseña `Cayala2026`.
- Las VLANs se crean únicamente en SW-Z2-P para evitar conflictos de número de revisión.

## 6. STP (Rapid PVST+)

| VLAN | Root bridge | Prioridad | Secundario (prioridad) | Justificación |
|---|---|---|---|---|
| 13 | SW-Z1-P | 4096 | SW-Z2-P (8192) | El principal de cada zona es root de su VLAN |
| 23 | SW-Z2-P | 4096 | - | Núcleo y gateway de la red |
| 33 | SW-Z3-P | 4096 | SW-Z2-P (8192) | Ídem |
| 43 | SW-Z4-P | 4096 | SW-Z2-P (8192) | Ídem |
| 53 | SW-Z5-P | 4096 | SW-Z2-P (8192) | Ídem |
| 99 | SW-Z2-P | 4096 | - | Administración centralizada en el núcleo |

## 7. EtherChannel (PAgP)

- Po1 entre SW-Z2-P y SW-Z1-P (Gi0/1-2 en ambos), modo `desirable`/`desirable`.
- Justificación: la Zona 1 tiene 60 hosts y tráfico alto de visitantes; se duplica el ancho de banda y se tiene redundancia ante la caída de un enlace.

## 8. Dominios de colisión y de broadcast

Todos los enlaces son full-duplex sobre switches, por lo que **cada puerto de switch en uso es un dominio de colisión**.

| Concepto | Cantidad |
|---|---|
| Enlaces switch-switch (el Po1 cuenta 2 enlaces físicos) | 12 |
| Enlaces switch-PC (3 por zona x 5) | 15 |
| **Total de dominios de colisión** | **27** |
| Dominios de broadcast (VLANs de zona) | 5 |

Los dominios de colisión los delimitan los puertos de los switches. Esta segmentación es adecuada porque elimina las colisiones y permite full-duplex.

## 9. Verificación (comandos usados)

`show vlan brief`, `show interfaces trunk`, `show vtp status`, `show spanning-tree summary`, `show spanning-tree vlan <id>`, `show etherchannel summary`, `show ip route`, `show ip interface brief`, `ping`.

## 10. Capturas de verificación

Todas las imágenes están en la carpeta `capturas/`.

### 10.1 Topología y cableado

**Topología completa en Packet Tracer**

![Topología completa en Packet Tracer](capturas/01_topologia_completa.png)

**Vecinos CDP de SW-Z2-P: el cableado coincide con el diseño**

![Vecinos CDP de SW-Z2-P: el cableado coincide con el diseño](capturas/02_cdp_vecinos_SW-Z2-P.png)

### 10.2 VLANs y VTP

**VLANs 13, 23, 33, 43, 53, 99 y 999 en el servidor SW-Z2-P**

![VLANs 13, 23, 33, 43, 53, 99 y 999 en el servidor SW-Z2-P](capturas/03_vlan_brief_SW-Z2-P.png)

**VTP en SW-Z2-P: modo servidor, dominio 201708993**

![VTP en SW-Z2-P: modo servidor, dominio 201708993](capturas/04_vtp_status_SW-Z2-P.png)

**VTP en SW-Z1-A: modo cliente, misma revisión que el servidor**

![VTP en SW-Z1-A: modo cliente, misma revisión que el servidor](capturas/05_vtp_status_SW-Z1-A.png)

**SW-Z1-A: Fa0/2-4 en VLAN 13, resto en Blackhole 999**

![SW-Z1-A: Fa0/2-4 en VLAN 13, resto en Blackhole 999](capturas/06_vlan_brief_SW-Z1-A.png)

**SW-Z2-A: Fa0/2-4 en VLAN 23, resto en Blackhole 999**

![SW-Z2-A: Fa0/2-4 en VLAN 23, resto en Blackhole 999](capturas/07_vlan_brief_SW-Z2-A.png)

**SW-Z3-A: Fa0/2-4 en VLAN 33, resto en Blackhole 999**

![SW-Z3-A: Fa0/2-4 en VLAN 33, resto en Blackhole 999](capturas/08_vlan_brief_SW-Z3-A.png)

**SW-Z4-A: Fa0/2-4 en VLAN 43, resto en Blackhole 999**

![SW-Z4-A: Fa0/2-4 en VLAN 43, resto en Blackhole 999](capturas/09_vlan_brief_SW-Z4-A.png)

**SW-Z5-A: Fa0/2-4 en VLAN 53, resto en Blackhole 999**

![SW-Z5-A: Fa0/2-4 en VLAN 53, resto en Blackhole 999](capturas/10_vlan_brief_SW-Z5-A.png)

### 10.3 Troncales

**Troncales de SW-Z2-P: nativa 99 y VLANs permitidas restringidas**

![Troncales de SW-Z2-P: nativa 99 y VLANs permitidas restringidas](capturas/11_trunk_SW-Z2-P.png)

**Troncal de SW-Z1-A hacia su switch principal**

![Troncal de SW-Z1-A hacia su switch principal](capturas/12_trunk_SW-Z1-A.png)

### 10.4 Rapid PVST+ (root bridge por VLAN)

**VLAN 13: SW-Z1-P es root bridge (prioridad 4096)**

![VLAN 13: SW-Z1-P es root bridge (prioridad 4096)](capturas/13_stp_vlan13_SW-Z1-P.png)

**VLAN 23: SW-Z2-P es root bridge (prioridad 4096)**

![VLAN 23: SW-Z2-P es root bridge (prioridad 4096)](capturas/14_stp_vlan23_SW-Z2-P.png)

**VLAN 33: SW-Z3-P es root bridge (prioridad 4096)**

![VLAN 33: SW-Z3-P es root bridge (prioridad 4096)](capturas/15_stp_vlan33_SW-Z3-P.png)

**VLAN 43: SW-Z4-P es root bridge (prioridad 4096)**

![VLAN 43: SW-Z4-P es root bridge (prioridad 4096)](capturas/16_stp_vlan43_SW-Z4-P.png)

**VLAN 53: SW-Z5-P es root bridge (prioridad 4096)**

![VLAN 53: SW-Z5-P es root bridge (prioridad 4096)](capturas/17_stp_vlan53_SW-Z5-P.png)

**VLAN 99: SW-Z2-P es root bridge (prioridad 4096)**

![VLAN 99: SW-Z2-P es root bridge (prioridad 4096)](capturas/18_stp_vlan99_SW-Z2-P.png)

### 10.5 EtherChannel (PAgP)

**Po1(SU) con Gig0/1(P) y Gig0/2(P) en SW-Z2-P**

![Po1(SU) con Gig0/1(P) y Gig0/2(P) en SW-Z2-P](capturas/19_etherchannel_SW-Z2-P.png)

**Po1(SU) con Gig0/1(P) y Gig0/2(P) en SW-Z1-P**

![Po1(SU) con Gig0/1(P) y Gig0/2(P) en SW-Z1-P](capturas/20_etherchannel_SW-Z1-P.png)

### 10.6 Enrutamiento entre VLANs

**show ip route en SW-Z2-P: cinco redes conectadas**

![show ip route en SW-Z2-P: cinco redes conectadas](capturas/21_ip_route_SW-Z2-P.png)

**Interfaces VLAN 13, 23, 33, 43 y 53 en up/up**

![Interfaces VLAN 13, 23, 33, 43 y 53 en up/up](capturas/22_ip_interface_brief_SW-Z2-P.png)

### 10.7 Conectividad dentro de cada zona (10 pings)

**Zona 1: 10 pings exitosos**

![Zona 1: 10 pings exitosos](capturas/23_ping_zona1.png)

**Zona 2: 10 pings exitosos**

![Zona 2: 10 pings exitosos](capturas/24_ping_zona2.png)

**Zona 3: 10 pings exitosos**

![Zona 3: 10 pings exitosos](capturas/25_ping_zona3.png)

**Zona 4: 10 pings exitosos**

![Zona 4: 10 pings exitosos](capturas/26_ping_zona4.png)

**Zona 5: 10 pings exitosos**

![Zona 5: 10 pings exitosos](capturas/27_ping_zona5.png)

### 10.8 Conectividad entre zonas

**Zona 1 a Zona 2**

![Zona 1 a Zona 2](capturas/28_ping_z1_a_z2.png)

**Zona 1 a Zona 3**

![Zona 1 a Zona 3](capturas/29_ping_z1_a_z3.png)

**Zona 1 a Zona 4**

![Zona 1 a Zona 4](capturas/30_ping_z1_a_z4.png)

**Zona 1 a Zona 5**

![Zona 1 a Zona 5](capturas/31_ping_z1_a_z5.png)

### 10.9 Prueba de redundancia

**Un enlace del EtherChannel apagado: Po1 sigue activo con el otro**

![Un enlace del EtherChannel apagado: Po1 sigue activo con el otro](capturas/32_redundancia_enlace_caido.png)

**La conectividad hacia la Zona 1 se mantiene con un enlace caído**

![La conectividad hacia la Zona 1 se mantiene con un enlace caído](capturas/33_redundancia_ping.png)

### 10.10 Análisis ARP e ICMP

**Modo simulación: ARP seguido de ICMP entre zonas**

![Modo simulación: ARP seguido de ICMP entre zonas](capturas/34_simulacion_arp_icmp.png)

## 11. Decisiones de diseño y alcance

- **Aislamiento y conectividad:** cada VLAN es un dominio de broadcast propio, por lo que las zonas están aisladas en capa 2. La conectividad entre zonas que pide el alcance de la práctica se resuelve en capa 3 con interfaces VLAN (SVI) en el switch multicapa SW-Z2-P (`ip routing`). Ese es el único punto por el que el tráfico cruza de una VLAN a otra y el que permite el `show ip route`. Restringir qué zonas pueden hablar entre sí (por ejemplo con ACL) queda fuera del alcance de esta práctica.
- **VTP:** las VLANs se crean únicamente en SW-Z2-P y los demás switches las reciben por VTP, para evitar conflictos de número de revisión entre servidores del mismo dominio.
- **Configuración en dos fases:** los clientes VTP solo pueden asignar puertos a VLANs que ya recibieron del servidor, por lo que cada script se aplica en dos partes (base y VTP primero; STP, SVIs, puertos de acceso y Blackhole después).
- **EtherChannel:** el orden de configuración importa. Los puertos miembros se agregan al `channel-group` sin configuración troncal y el trunk (nativa 99 y VLANs permitidas) se configura en la interfaz `port-channel`.
- **Scripts:** los comandos de cada switch están en `scripts/`, un archivo por dispositivo.
