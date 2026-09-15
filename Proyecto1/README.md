# Reporte Técnico de Configuración de Red Jerárquica

**Estudiante:** Mariano Roberto Rac Noguera
**Carné:** 202101149
**Universidad:** Universidad de San Carlos de Guatemala (USAC)
**Facultad:** Facultad de Ingeniería (FIUSAC)
**Escuela:** Escuela de Ciencias y Sistemas
**Curso:** Redes de Computadoras 1
**Catedrático:** Ing. Pedro Pablo Hernández
**Auxiliar:** Adrián Fernández
**Proyecto:** SmartCity Tech Park, Proyecto 1, Segundo Semestre 2026

---

## 1. Introducción y objetivos

Este informe documenta el diseño, la implementación y la verificación de una arquitectura de red jerárquica para el complejo SmartCity Tech Park, desarrollada en Cisco Packet Tracer. La topología está estructurada en las capas de Núcleo (Centro de Datos), Distribución (SW-CORP-DIST y SW-ID-1) y Acceso (el resto de los switches del campus), interconectando cuatro áreas del negocio, Centro de Datos, Centro de I+D, Edificio Corporativo y Planta de Producción.

El proyecto cubre tres frentes técnicos principales. El primero es la sincronización centralizada de VLANs mediante VTP (VLAN Trunking Protocol), que simplifica la creación y propagación de VLANs a través de todos los switches del dominio sin necesidad de configurarlas manualmente en cada uno. El segundo es la agregación de enlaces mediante PAgP (Port Aggregation Protocol), que conforma los ocho enlaces EtherChannel del campus, incrementando el ancho de banda disponible y garantizando tolerancia a fallos ante la caída de un cable individual. El tercero es la estandarización de los enlaces troncales bajo IEEE 802.1Q con la VLAN nativa 99, usada para el transporte de tráfico de control sin etiquetar entre switches.

## 2. Parámetros de diseño derivados del carné

El carné 202101149 termina en el dígito 9 (impar) y su penúltimo dígito es 4. De ahí se derivan los siguientes parámetros obligatorios del proyecto.

| Parámetro | Valor asignado | Descripción técnica |
| :--- | :--- | :--- |
| Dominio VTP | Smart_4 | Identificador único del dominio de propagación VTP |
| Contraseña VTP | proyecto12S2026 | Clave requerida para que el resumen MD5 de los anuncios VTP coincida entre switches y se acepte la sincronización |
| Modo VTP en SW-CORE | Server | Único switch con permisos de creación y modificación de la base de datos de VLANs; los siete restantes operan en modo Client |
| VLAN nativa | 99 | VLAN designada para tráfico no etiquetado en todos los troncales del campus |
| Protocolo EtherChannel | PAgP | Protocolo propietario de Cisco para negociación dinámica de canales lógicos, asignado por tener carné impar |
| Protocolo de árbol de expansión | Rapid-PVST | Asignado por tener carné impar |
| Banner MOTD en switches de distribución | Acceso Restringido - TechPark_202101149 | Aplicado en SW-CORP-DIST y SW-ID-1, los dos switches que cumplen rol de distribución en la jerarquía |

Las cinco VLANs de negocio, creadas una sola vez en el switch servidor (SW-CORE) y propagadas por VTP a los demás switches del dominio, quedaron así.

| VLAN | Nombre | Ubicación física |
| :--- | :--- | :--- |
| 19 | GERENCIA | Edificio Corporativo |
| 29 | INVESTIGACION | Centro de I+D |
| 39 | PRODUCCION | Planta de Producción |
| 49 | SERVIDORES | Centro de Datos |
| 59 | VISITANTES | Edificio Corporativo (Área Común / Access Point) |

## 3. Diseño de la topología por área

![Topología completa del campus SmartCity Tech Park](Images/Screenshot%202026-09-14%20at%205.30.29%20PM.png)

### 3.1 Centro de Datos (Core)

SW-CORE actúa como núcleo de la red y como único switch en modo VTP Server del dominio Smart_4. Concentra el acceso a la granja de cuatro servidores críticos (Server-1 a Server-4, VLAN 49) y se interconecta mediante tres enlaces troncales redundantes (EtherChannel) hacia los tres switches de distribución y acceso restantes del campus.

Cada uno de los cuatro servidores se conecta al Core mediante un enlace físico individual e independiente de los demás. Esta decisión se justifica en que el enunciado deja explícitamente a criterio del estudiante la forma de resolver la interconexión de la granja, y bajo este diseño la falla de cualquiera de los cuatro cables únicamente afecta al servidor asociado a ese cable, sin comprometer la disponibilidad de los demás ni el acceso a la granja como conjunto.

### 3.2 Centro de I+D

Los switches SW-ID-1, SW-ID-2 y SW-ID-3 forman una malla completa (los tres interconectados entre sí), de modo que la caída de cualquiera de los tres no aísla a los demás del resto del campus. SW-ID-1 concentra la salida de toda el área hacia el Core mediante el enlace de mayor prioridad de diseño, ya que el enunciado exige que su troncal hacia el Centro de Datos soporte una mayor demanda de ancho de banda que el resto de los enlaces del campus. El área aloja ocho estaciones de trabajo repartidas entre los tres switches.

![Topología del Centro de I+D, malla SW-ID-1, SW-ID-2 y SW-ID-3](Images/Screenshot%202026-09-14%20at%204.50.51%20PM.png)

### 3.3 Edificio Corporativo

SW-CORP-DIST distribuye el tráfico hacia dos alas físicas (SW-ALA-A y SW-ALA-B), cada una con su propio switch de acceso. Además del enlace de cada ala hacia la distribución, existe un enlace directo entre SW-ALA-A y SW-ALA-B, de modo que la conectividad entre ambas alas se mantiene activa aunque falle la ruta hacia SW-CORP-DIST. El Área Común para visitantes, servida por un Access Point inalámbrico conectado a SW-ALA-B, queda en la VLAN 59, completamente aislada a nivel de dominio de broadcast del resto del tráfico corporativo (VLAN 19, Gerencia), aunque comparta el mismo switch físico.

![Topología del Edificio Corporativo, SW-CORE y SW-CORP-DIST](Images/Screenshot%202026-09-14%20at%2010.10.26%20AM.png)

### 3.4 Planta de Producción

SW-PROD-ACCESO conecta un Hub que agrupa las máquinas industriales heredadas (segmento Legacy), evidenciando un único dominio de colisión compartido en Capa 1 para ese segmento, tal como exige el enunciado. El switch se conecta al Core mediante un EtherChannel redundante.

## 4. Configuración aplicada por dispositivo

Esta sección documenta los comandos finales realmente aplicados en cada switch, ya verificados y funcionando, tras el proceso de depuración descrito en la sección 12.

### 4.1 Switch central (SW-CORE)

Se estableció el modo Server para VTP con el dominio y la contraseña del proyecto, se crearon y renombraron las cinco VLANs de negocio en mayúsculas según exige el enunciado, y se configuraron los tres EtherChannel hacia los switches de distribución y acceso, además de los puertos de acceso de los cuatro servidores.

```cisco
SW-CORE# configure terminal
SW-CORE(config)# hostname SW-CORE
SW-CORE(config)# vtp domain Smart_4
SW-CORE(config)# vtp password proyecto12S2026
SW-CORE(config)# vlan 19
SW-CORE(config-vlan)# name GERENCIA
SW-CORE(config-vlan)# vlan 29
SW-CORE(config-vlan)# name INVESTIGACION
SW-CORE(config-vlan)# vlan 39
SW-CORE(config-vlan)# name PRODUCCION
SW-CORE(config-vlan)# vlan 49
SW-CORE(config-vlan)# name SERVIDORES
SW-CORE(config-vlan)# vlan 59
SW-CORE(config-vlan)# name VISITANTES
SW-CORE(config-vlan)# vlan 99
SW-CORE(config-vlan)# name NATIVA
SW-CORE(config-vlan)# exit
SW-CORE(config)# spanning-tree mode rapid-pvst
SW-CORE(config)# spanning-tree vlan 19,49,99 priority 24576

! Enlace hacia SW-ID-1 (Po1)
SW-CORE(config)# interface range fastEthernet0/11 - 12
SW-CORE(config-if-range)# switchport trunk native vlan 99
SW-CORE(config-if-range)# switchport mode trunk
SW-CORE(config-if-range)# channel-protocol pagp
SW-CORE(config-if-range)# channel-group 1 mode desirable
SW-CORE(config-if-range)# exit

! Enlace hacia SW-CORP-DIST (Po2)
SW-CORE(config)# interface range fastEthernet0/13 - 14
SW-CORE(config-if-range)# switchport trunk native vlan 99
SW-CORE(config-if-range)# switchport mode trunk
SW-CORE(config-if-range)# channel-protocol pagp
SW-CORE(config-if-range)# channel-group 2 mode desirable
SW-CORE(config-if-range)# exit

! Enlace hacia SW-PROD-ACCESO (Po3)
SW-CORE(config)# interface range fastEthernet0/5 - 6
SW-CORE(config-if-range)# switchport trunk native vlan 99
SW-CORE(config-if-range)# switchport mode trunk
SW-CORE(config-if-range)# channel-protocol pagp
SW-CORE(config-if-range)# channel-group 3 mode desirable
SW-CORE(config-if-range)# exit

! Puertos de acceso de la granja de servidores (VLAN 49)
SW-CORE(config)# interface range fastEthernet0/7 - 10
SW-CORE(config-if-range)# switchport access vlan 49
SW-CORE(config-if-range)# switchport mode access
SW-CORE(config-if-range)# no cdp enable
SW-CORE(config-if-range)# end
```

### 4.2 Centro de I+D (SW-ID-1, SW-ID-2, SW-ID-3)

SW-ID-1 concentra tres EtherChannel, hacia el Core, hacia SW-ID-2 y hacia SW-ID-3, además del banner de seguridad y la prioridad de Root Bridge para la VLAN 29.

```cisco
SW-ID-1# configure terminal
SW-ID-1(config)# hostname SW-ID-1
SW-ID-1(config)# vtp mode client
SW-ID-1(config)# vtp password proyecto12S2026
SW-ID-1(config)# banner motd #Acceso Restringido - TechPark_202101149#
SW-ID-1(config)# spanning-tree mode rapid-pvst
SW-ID-1(config)# spanning-tree vlan 29 priority 24576

! Enlace hacia SW-CORE (Po1)
SW-ID-1(config)# interface range fastEthernet0/6 - 7
SW-ID-1(config-if-range)# switchport trunk native vlan 99
SW-ID-1(config-if-range)# switchport mode trunk
SW-ID-1(config-if-range)# channel-protocol pagp
SW-ID-1(config-if-range)# channel-group 1 mode auto
SW-ID-1(config-if-range)# exit

! Enlace hacia SW-ID-2 (Po2)
SW-ID-1(config)# interface range fastEthernet0/4 - 5
SW-ID-1(config-if-range)# switchport trunk native vlan 99
SW-ID-1(config-if-range)# switchport mode trunk
SW-ID-1(config-if-range)# channel-protocol pagp
SW-ID-1(config-if-range)# channel-group 2 mode desirable
SW-ID-1(config-if-range)# exit

! Enlace hacia SW-ID-3 (Po3)
SW-ID-1(config)# interface range fastEthernet0/8 - 9
SW-ID-1(config-if-range)# switchport trunk native vlan 99
SW-ID-1(config-if-range)# switchport mode trunk
SW-ID-1(config-if-range)# channel-protocol pagp
SW-ID-1(config-if-range)# channel-group 3 mode desirable
SW-ID-1(config-if-range)# exit

! Puertos de acceso de estaciones de trabajo (VLAN 29)
SW-ID-1(config)# interface range fastEthernet0/1 - 3
SW-ID-1(config-if-range)# switchport mode access
SW-ID-1(config-if-range)# switchport access vlan 29
SW-ID-1(config-if-range)# exit
SW-ID-1(config)# interface fastEthernet0/10
SW-ID-1(config-if)# switchport mode access
SW-ID-1(config-if)# switchport access vlan 29
SW-ID-1(config-if)# end
```

SW-ID-2 forma parte de la malla con dos EtherChannel, hacia SW-ID-1 y hacia SW-ID-3.

```cisco
SW-ID-2# configure terminal
SW-ID-2(config)# hostname SW-ID-2
SW-ID-2(config)# vtp mode client
SW-ID-2(config)# vtp password proyecto12S2026

! Enlace hacia SW-ID-1 (Po1)
SW-ID-2(config)# interface range fastEthernet0/6 - 7
SW-ID-2(config-if-range)# switchport trunk native vlan 99
SW-ID-2(config-if-range)# switchport mode trunk
SW-ID-2(config-if-range)# channel-protocol pagp
SW-ID-2(config-if-range)# channel-group 1 mode auto
SW-ID-2(config-if-range)# exit

! Enlace hacia SW-ID-3 (Po2)
SW-ID-2(config)# interface range fastEthernet0/4 - 5
SW-ID-2(config-if-range)# switchport trunk native vlan 99
SW-ID-2(config-if-range)# switchport mode trunk
SW-ID-2(config-if-range)# channel-protocol pagp
SW-ID-2(config-if-range)# channel-group 2 mode desirable
SW-ID-2(config-if-range)# exit

! Puertos de acceso (VLAN 29)
SW-ID-2(config)# interface range fastEthernet0/1 - 3
SW-ID-2(config-if-range)# switchport mode access
SW-ID-2(config-if-range)# switchport access vlan 29
SW-ID-2(config-if-range)# end
```

SW-ID-3 cierra la malla con dos EtherChannel, hacia SW-ID-2 (sobre puertos Gigabit) y hacia SW-ID-1.

```cisco
SW-ID-3# configure terminal
SW-ID-3(config)# hostname SW-ID-3
SW-ID-3(config)# vtp mode client
SW-ID-3(config)# vtp password proyecto12S2026

! Enlace hacia SW-ID-2 (Po1)
SW-ID-3(config)# interface range gigabitEthernet0/1 - 2
SW-ID-3(config-if-range)# switchport trunk native vlan 99
SW-ID-3(config-if-range)# switchport mode trunk
SW-ID-3(config-if-range)# channel-protocol pagp
SW-ID-3(config-if-range)# channel-group 1 mode auto
SW-ID-3(config-if-range)# exit

! Enlace hacia SW-ID-1 (Po2)
SW-ID-3(config)# interface range fastEthernet0/3 - 4
SW-ID-3(config-if-range)# switchport trunk native vlan 99
SW-ID-3(config-if-range)# switchport mode trunk
SW-ID-3(config-if-range)# channel-protocol pagp
SW-ID-3(config-if-range)# channel-group 2 mode auto
SW-ID-3(config-if-range)# exit

! Puertos de acceso (VLAN 29)
SW-ID-3(config)# interface range fastEthernet0/1 - 2
SW-ID-3(config-if-range)# switchport mode access
SW-ID-3(config-if-range)# switchport access vlan 29
SW-ID-3(config-if-range)# end
```

### 4.3 Edificio Corporativo (SW-CORP-DIST, SW-ALA-A, SW-ALA-B)

SW-CORP-DIST concentra el banner de seguridad, la prioridad de Root Bridge para la VLAN 59, y tres EtherChannel, hacia el Core y hacia cada una de las dos alas.

```cisco
SW-CORP-DIST# configure terminal
SW-CORP-DIST(config)# hostname SW-CORP-DIST
SW-CORP-DIST(config)# vtp mode client
SW-CORP-DIST(config)# vtp password proyecto12S2026
SW-CORP-DIST(config)# banner motd #Acceso Restringido - TechPark_202101149#
SW-CORP-DIST(config)# spanning-tree mode rapid-pvst
SW-CORP-DIST(config)# spanning-tree vlan 59 priority 24576

! Enlace hacia SW-CORE (Po1)
SW-CORP-DIST(config)# interface range fastEthernet0/7 - 8
SW-CORP-DIST(config-if-range)# switchport trunk native vlan 99
SW-CORP-DIST(config-if-range)# switchport mode trunk
SW-CORP-DIST(config-if-range)# channel-protocol pagp
SW-CORP-DIST(config-if-range)# channel-group 1 mode auto
SW-CORP-DIST(config-if-range)# exit

! Enlace hacia SW-ALA-A (Po2)
SW-CORP-DIST(config)# interface fastEthernet0/2
SW-CORP-DIST(config-if)# switchport trunk native vlan 99
SW-CORP-DIST(config-if)# switchport mode trunk
SW-CORP-DIST(config-if)# channel-protocol pagp
SW-CORP-DIST(config-if)# channel-group 2 mode desirable
SW-CORP-DIST(config-if)# interface fastEthernet0/5
SW-CORP-DIST(config-if)# switchport trunk native vlan 99
SW-CORP-DIST(config-if)# switchport mode trunk
SW-CORP-DIST(config-if)# channel-protocol pagp
SW-CORP-DIST(config-if)# channel-group 2 mode desirable
SW-CORP-DIST(config-if)# exit

! Enlace hacia SW-ALA-B (Po3)
SW-CORP-DIST(config)# interface fastEthernet0/3
SW-CORP-DIST(config-if)# switchport trunk native vlan 99
SW-CORP-DIST(config-if)# switchport mode trunk
SW-CORP-DIST(config-if)# channel-protocol pagp
SW-CORP-DIST(config-if)# channel-group 3 mode desirable
SW-CORP-DIST(config-if)# interface fastEthernet0/6
SW-CORP-DIST(config-if)# switchport trunk native vlan 99
SW-CORP-DIST(config-if)# switchport mode trunk
SW-CORP-DIST(config-if)# channel-protocol pagp
SW-CORP-DIST(config-if)# channel-group 3 mode desirable
SW-CORP-DIST(config-if)# end
```

SW-ALA-A conecta las estaciones de Gerencia y forma dos EtherChannel, hacia SW-CORP-DIST y hacia SW-ALA-B.

```cisco
SW-ALA-A# configure terminal
SW-ALA-A(config)# hostname SW-ALA-A
SW-ALA-A(config)# vtp mode client
SW-ALA-A(config)# vtp password proyecto12S2026

! Enlace hacia SW-ALA-B (Po1)
SW-ALA-A(config)# interface range fastEthernet0/5,fastEthernet0/7
SW-ALA-A(config-if-range)# switchport trunk native vlan 99
SW-ALA-A(config-if-range)# switchport mode trunk
SW-ALA-A(config-if-range)# channel-protocol pagp
SW-ALA-A(config-if-range)# channel-group 1 mode desirable
SW-ALA-A(config-if-range)# exit

! Enlace hacia SW-CORP-DIST (Po2)
SW-ALA-A(config)# interface range fastEthernet0/4,fastEthernet0/6
SW-ALA-A(config-if-range)# switchport trunk native vlan 99
SW-ALA-A(config-if-range)# switchport mode trunk
SW-ALA-A(config-if-range)# channel-protocol pagp
SW-ALA-A(config-if-range)# channel-group 2 mode auto
SW-ALA-A(config-if-range)# exit

! Puertos de acceso (VLAN 19)
SW-ALA-A(config)# interface range fastEthernet0/1 - 3
SW-ALA-A(config-if-range)# switchport mode access
SW-ALA-A(config-if-range)# switchport access vlan 19
SW-ALA-A(config-if-range)# end
```

SW-ALA-B conecta las estaciones de Gerencia además del Access Point de Visitantes, y forma los mismos dos EtherChannel que SW-ALA-A.

```cisco
SW-ALA-B# configure terminal
SW-ALA-B(config)# hostname SW-ALA-B
SW-ALA-B(config)# vtp mode client
SW-ALA-B(config)# vtp password proyecto12S2026

! Enlace hacia SW-ALA-A (Po1)
SW-ALA-B(config)# interface range fastEthernet0/5,fastEthernet0/7
SW-ALA-B(config-if-range)# switchport trunk native vlan 99
SW-ALA-B(config-if-range)# switchport mode trunk
SW-ALA-B(config-if-range)# channel-protocol pagp
SW-ALA-B(config-if-range)# channel-group 1 mode auto
SW-ALA-B(config-if-range)# exit

! Enlace hacia SW-CORP-DIST (Po2)
SW-ALA-B(config)# interface range fastEthernet0/4,fastEthernet0/6
SW-ALA-B(config-if-range)# switchport trunk native vlan 99
SW-ALA-B(config-if-range)# switchport mode trunk
SW-ALA-B(config-if-range)# channel-protocol pagp
SW-ALA-B(config-if-range)# channel-group 2 mode auto
SW-ALA-B(config-if-range)# exit

! Puerto hacia el Access Point (VLAN 59, Visitantes, aislada)
SW-ALA-B(config)# interface fastEthernet0/1
SW-ALA-B(config-if)# switchport mode access
SW-ALA-B(config-if)# switchport access vlan 59
SW-ALA-B(config-if)# exit

! Puertos de acceso de Gerencia (VLAN 19)
SW-ALA-B(config)# interface range fastEthernet0/2 - 3
SW-ALA-B(config-if-range)# switchport mode access
SW-ALA-B(config-if-range)# switchport access vlan 19
SW-ALA-B(config-if-range)# end
```

### 4.4 Planta de Producción (SW-PROD-ACCESO)

SW-PROD-ACCESO conecta el segmento Legacy a través de un Hub y mantiene un único EtherChannel hacia el Core, además de la prioridad de Root Bridge para la VLAN 39.

```cisco
SW-PROD-ACCESO# configure terminal
SW-PROD-ACCESO(config)# hostname SW-PROD-ACCESO
SW-PROD-ACCESO(config)# vtp mode client
SW-PROD-ACCESO(config)# vtp password proyecto12S2026
SW-PROD-ACCESO(config)# spanning-tree mode rapid-pvst
SW-PROD-ACCESO(config)# spanning-tree vlan 39 priority 24576

! Enlace hacia SW-CORE (Po1)
SW-PROD-ACCESO(config)# interface range fastEthernet0/2 - 3
SW-PROD-ACCESO(config-if-range)# switchport trunk native vlan 99
SW-PROD-ACCESO(config-if-range)# switchport mode trunk
SW-PROD-ACCESO(config-if-range)# channel-protocol pagp
SW-PROD-ACCESO(config-if-range)# channel-group 1 mode auto
SW-PROD-ACCESO(config-if-range)# exit

! Puerto hacia el Hub del segmento Legacy (VLAN 39)
SW-PROD-ACCESO(config)# interface fastEthernet0/1
SW-PROD-ACCESO(config-if)# switchport mode access
SW-PROD-ACCESO(config-if)# switchport access vlan 39
SW-PROD-ACCESO(config-if)# end
```

## 5. EtherChannel, resumen de los ocho enlaces del campus

| Enlace | Extremo (modo) e interfaces | Extremo (modo) e interfaces |
| :--- | :--- | :--- |
| Core, I+D | SW-CORE (desirable) Fa0/11, Fa0/12 | SW-ID-1 (auto) Fa0/6, Fa0/7 |
| Core, Corporativo | SW-CORE (desirable) Fa0/13, Fa0/14 | SW-CORP-DIST (auto) Fa0/7, Fa0/8 |
| Core, Producción | SW-CORE (desirable) Fa0/5, Fa0/6 | SW-PROD-ACCESO (auto) Fa0/2, Fa0/3 |
| I+D 1 a I+D 2 | SW-ID-1 (desirable) Fa0/4, Fa0/5 | SW-ID-2 (auto) Fa0/6, Fa0/7 |
| I+D 1 a I+D 3 | SW-ID-1 (desirable) Fa0/8, Fa0/9 | SW-ID-3 (auto) Fa0/3, Fa0/4 |
| I+D 2 a I+D 3 | SW-ID-2 (desirable) Fa0/4, Fa0/5 | SW-ID-3 (auto) Gig0/1, Gig0/2 |
| Corp-Dist a Ala-A | SW-CORP-DIST (desirable) Fa0/2, Fa0/5 | SW-ALA-A (auto) Fa0/4, Fa0/6 |
| Corp-Dist a Ala-B | SW-CORP-DIST (desirable) Fa0/3, Fa0/6 | SW-ALA-B (auto) Fa0/4, Fa0/6 |
| Ala-A a Ala-B | SW-ALA-A (desirable) Fa0/5, Fa0/7 | SW-ALA-B (auto) Fa0/5, Fa0/7 |

Nota técnica de diseño, verificada durante la implementación, un mismo EtherChannel no puede mezclar puertos FastEthernet con puertos GigabitEthernet aunque se fuerce manualmente la misma velocidad y dúplex en ambos, ya que Cisco IOS los trata como tipos de interfaz distintos e incompatibles para el mismo canal. Por esa razón, cualquier enlace redundante del proyecto usa el mismo tipo de puerto en ambos extremos.

## 6. VTP

SW-CORE opera en modo Server y es la única fuente de verdad para la base de datos de VLANs del dominio Smart_4. Los siete switches restantes operan en modo Client, reciben la base de datos de VLANs automáticamente y no permiten crear ni modificar VLANs de forma local. La contraseña del dominio, proyecto12S2026, está configurada de forma idéntica en los ocho switches, requisito indispensable para que el resumen criptográfico (MD5 digest) de los anuncios coincida y la sincronización se acepte.

## 7. Rapid-PVST y Root Bridge

Cada VLAN calcula su propio árbol de expansión, con el Root Bridge repartido según el switch que concentra el tráfico más relevante para esa VLAN, en vez de dejarlo siempre en el mismo dispositivo.

| VLAN | Root Bridge | Justificación |
| :--- | :--- | :--- |
| 19, Gerencia | SW-CORE | Tráfico administrativo que constantemente necesita llegar a los servicios centrales del campus |
| 29, Investigación | SW-ID-1 | Switch que concentra la salida de toda el área de I+D hacia el resto del campus |
| 39, Producción | SW-PROD-ACCESO | Único switch del área, punto natural de referencia para su tráfico |
| 49, Servidores | SW-CORE | Tráfico de mayor volumen hacia la granja de servidores, conviene anclarlo al núcleo |
| 59, Visitantes | SW-CORP-DIST | Tráfico que el enunciado exige mantener aislado y sin necesidad de ruta óptima hacia el Core |
| 99, nativa | SW-CORE | VLAN que transporta el tráfico de control entre switches, anclada al núcleo |

## 8. Seguridad básica

El banner MOTD "Acceso Restringido - TechPark_202101149" está configurado en los dos switches que cumplen el rol estructural de distribución dentro de la jerarquía, SW-CORP-DIST (agrega el tráfico de las dos alas del Corporativo hacia el Core) y SW-ID-1 (agrega el tráfico de SW-ID-2 y SW-ID-3 hacia el Core). La VLAN nativa 99 está configurada de forma consistente en todos los enlaces troncales del campus.

## 9. Direccionamiento IP

Se utilizó el rango privado 172.16.X.0/24, donde X coincide con el número de la VLAN, para que la relación entre VLAN y subred sea evidente. La máscara /24 reserva la primera dirección de cada rango como dirección de red y la última como dirección de broadcast, dejando 254 direcciones utilizables para hosts.

| VLAN | Red | Dispositivos |
| :--- | :--- | :--- |
| 19, Gerencia | 172.16.19.0/24 | Cinco PCs del Edificio Corporativo (Ala-A y Ala-B) |
| 29, Investigación | 172.16.29.0/24 | Ocho estaciones de trabajo del Centro de I+D |
| 39, Producción | 172.16.39.0/24 | Tres PCs del segmento Legacy en Planta de Producción |
| 49, Servidores | 172.16.49.0/24 | Cuatro servidores del Centro de Datos |
| 59, Visitantes | 172.16.59.0/24 | Dos laptops inalámbricas conectadas al Access Point |

No se configuró puerta de enlace en ningún dispositivo, ya que la topología es exclusivamente de Capa 1 y Capa 2, sin ningún dispositivo de Capa 3 que enrute entre VLANs. Esto es intencional y explica el resultado esperado de las pruebas de conectividad entre VLANs distintas descritas a continuación.

## 10. Dominios de colisión

_Pendiente de completar con el conteo exacto de puertos activos por switch, verificado con `show interfaces status` en cada dispositivo. Cada puerto activo de un switch constituye su propio dominio de colisión, salvo el segmento Legacy de Producción, donde el Hub agrupa a todos sus dispositivos conectados en un único dominio de colisión compartido._

| Switch o segmento | Número de dominios de colisión que genera | Detalle |
| :--- | :--- | :--- |
| SW-CORE | | Un dominio por cada puerto activo (servidores y enlaces troncales) |
| SW-ID-1 | | |
| SW-ID-2 | | |
| SW-ID-3 | | |
| SW-CORP-DIST | | |
| SW-ALA-A | | |
| SW-ALA-B | | |
| SW-PROD-ACCESO | | Puertos activos hacia el Core y hacia el Hub |
| Segmento Legacy (Hub0) | 1 (compartido) | Todas las PCs conectadas al Hub comparten un único dominio de colisión, tal como exige el enunciado; el switch de acceso solo contiene y aísla ese dominio del resto de la red, sin eliminarlo |

## 11. Dominios de broadcast

Cada VLAN activa constituye un dominio de broadcast independiente, delimitado por los switches del campus y propagado por los enlaces troncales.

| VLAN | Dominio de broadcast |
| :--- | :--- |
| 19, Gerencia | Todas las PCs de Gerencia en el Edificio Corporativo |
| 29, Investigación | Las ocho estaciones de trabajo del Centro de I+D |
| 39, Producción | Las PCs del segmento Legacy en Planta de Producción |
| 49, Servidores | Los cuatro servidores del Centro de Datos |
| 59, Visitantes | Las laptops conectadas al Access Point |
| 99, nativa | Tráfico de control sin etiquetar entre switches, no transporta tráfico de usuario |

## 12. Pruebas de conectividad

Se realizaron pruebas de ping dentro de cada VLAN (deben ser exitosas) y entre VLANs distintas (deben fallar, ya que no existe un dispositivo de Capa 3 que enrute entre ellas). Un ping fallido entre VLANs no es un error del diseño, es la evidencia esperada de que la segmentación lógica está funcionando correctamente.

_Pendiente de insertar las capturas de pantalla correspondientes a cada prueba._

Pruebas exitosas dentro de la misma VLAN, entre las tres PCs de Producción (VLAN 39), entre las ocho estaciones de I+D atravesando la malla redundante de tres switches (VLAN 29), entre PCs de Gerencia en switches distintos del Corporativo (VLAN 19), entre las dos laptops conectadas al Access Point (VLAN 59), y entre los cuatro servidores del Centro de Datos (VLAN 49).

Pruebas fallidas entre VLANs distintas, desde una laptop de Visitantes hacia una PC de Gerencia, y desde un servidor hacia una PC de Producción.

![Ping exitoso dentro de VLAN 39, Producción, entre PC6 y PC7](Images/Screenshot%202026-09-14%20at%204.29.10%20PM.png)

![Ping exitoso dentro de VLAN 29, Centro de I+D, atravesando la malla redundante](Images/Screenshot%202026-09-14%20at%205.19.54%20PM.png)

![Ping exitoso dentro de VLAN 59, Visitantes, entre las dos laptops, y ping fallido desde una laptop de Visitantes hacia una PC de Gerencia (VLAN 19)](Images/Screenshot%202026-09-14%20at%205.28.08%20PM.png)

![Ping exitoso dentro de VLAN 49, Servidores, y ping fallido desde un servidor hacia una PC de Producción (VLAN 39)](Images/Screenshot%202026-09-14%20at%205.30.34%20PM.png)

## 13. Comandos de verificación (evidencia para el manual)

_Pendiente de insertar capturas de pantalla de los siguientes comandos, ejecutados en los switches relevantes._

Salida de `show spanning-tree`, `show etherchannel summary` y `show interfaces trunk` en SW-CORE, mostrando los tres Port-channel activos (SU) y las seis VLANs forwarding. Salida de `show vtp status` en SW-CORE y en al menos un switch cliente, mostrando el mismo número de revisión y el mismo MD5 digest. Pruebas de ping exitosas y fallidas de la sección 12.

## 14. Presupuesto estimado

_Pendiente de completar con precios de referencia de equipos físicos simulados._

| Equipo | Cantidad | Precio unitario estimado | Subtotal |
| :--- | :--- | :--- | :--- |
| Switch Cisco Catalyst 2960-24TT | 8 | | |
| Hub de 4 u 8 puertos | 1 | | |
| Access Point | 1 | | |
| Módulos de fibra óptica (si se justifican en algún enlace) | | | |
| Cableado UTP Cat5e/Cat6 | | | |
| Cableado de fibra óptica (si aplica) | | | |
| **Total** | | | |

## 15. Incidencias resueltas durante la implementación

Este apartado documenta el proceso de diagnóstico y resolución de problemas encontrados durante la configuración, como evidencia de la aplicación práctica de los conceptos del curso.

El primer EtherChannel configurado (Core hacia I+D) no lograba pasar de estado individual a bundled a pesar de que la configuración lógica de VLAN nativa, modo trunk y channel-group coincidía en ambos extremos. El diagnóstico con `show cdp neighbors` reveló que los puertos configurados inicialmente en el Core en realidad estaban cableados hacia otro dispositivo, no hacia SW-ID-1, cuyo enlace real terminó usando los puertos Fa0/11 y Fa0/12. La lección aplicada fue verificar siempre la adyacencia real por CDP antes de perseguir un supuesto problema de negociación PAgP.

En más de una ocasión, un EtherChannel con configuración lógica idéntica en ambos lados quedó en estado individual sin causa aparente. Se determinó que Packet Tracer, tras múltiples cambios sucesivos de configuración sobre las mismas interfaces, puede dejar el proceso interno de negociación PAgP en un estado inconsistente que ni un ciclo de shutdown y no shutdown ni la remoción y reconstrucción del channel-group logran limpiar; cerrar y volver a abrir el archivo del proyecto resultó ser el método consistentemente efectivo para restablecer la negociación.

Se detectó que reutilizar el mismo número de channel-group para dos enlaces físicos distintos hacia vecinos distintos en un mismo switch provoca que el switch solo pueda formar el canal con uno de los dos vecinos, dejando al otro con puertos sueltos fuera de cualquier agregación. La corrección consistió en asignar un número de channel-group distinto por cada vecino en cada switch, recordando que ese número es de significado local y no necesita coincidir entre los dos extremos de un enlace.

Se determinó, mediante pruebas con velocidad y dúplex forzados manualmente, que un EtherChannel no puede mezclar un puerto FastEthernet con uno GigabitEthernet, restricción real de Cisco IOS y no una limitación del simulador.

La propagación de VTP no se completaba a pesar de que el dominio y el modo Client estaban correctamente configurados; el conteo de `show vtp counters` mostró un alto número de errores de digest, causado por la ausencia de la contraseña VTP en el switch servidor, la cual nunca se había configurado. Al establecer la contraseña de forma idéntica en los ocho switches, la sincronización se completó de inmediato.

Finalmente, varios switches de acceso (SW-ID-2, SW-ID-3 y SW-ALA-B) tenían sus interfaces de acceso hacia las PCs y el Access Point aún en la VLAN 1 por defecto, nunca asignadas a la VLAN de negocio correspondiente, lo cual causaba fallos de conectividad entre dispositivos que en apariencia debían estar en la misma red. Esto reforzó la importancia de verificar tanto la configuración del trunk como la del puerto de acceso final antes de asumir un problema de enrutamiento o de VLAN a nivel de switch intermedio.