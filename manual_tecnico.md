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

> Verificar los nombres en https://cayala.com/ antes de entregar.

| Zona | Establecimientos | Rol operativo | Hosts | Tráfico | Criticidad | Enlaces |
|---|---|---|---|---|---|---|
| 1 | Paseo Cayalá (tiendas) | Corazón comercial; visitantes y comercios | 60 | Alto | Alta | 3 (Po1 de 2 enlaces + 1 a Z4) |
| 2 | Oficinas y administración central | Administración de la red, oficinas y seguridad; aloja el enrutamiento | 28 | Medio | Crítica | 5 (núcleo) |
| 3 | Clínicas, laboratorios y bancos | Servicios médicos y bancarios | 12 | Bajo/Medio | Media | 2 (Z2 y Z4) |
| 4 | Restaurantes, cafés y cine | Gastronomía y entretenimiento | 50 | Alto | Alta | 3 (Z2, Z1, Z3) |
| 5 | Gimnasio, academias y servicios complementarios | Servicios secundarios | 7 | Bajo | Baja | 1 (Z2) |

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
- Los modelos 3560/2960 de Packet Tracer no tienen puertos de fibra; confirmar con el tutor si se exige fibra.

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

## 10. Capturas

Guardar en `capturas/` y enlazar aquí:

- [ ] Topología completa en Packet Tracer
- [ ] `show vlan brief` (clientes y servidores)
- [ ] `show vtp status`
- [ ] `show interfaces trunk`
- [ ] `show spanning-tree` (root de cada VLAN)
- [ ] `show etherchannel summary`
- [ ] `show ip route` en SW-Z2-P
- [ ] 10 pings exitosos por zona
- [ ] Pings entre zonas
- [ ] Prueba de redundancia (apagar un enlace y repetir ping)
