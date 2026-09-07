# Reporte Técnico de Configuración de Red Jerárquica

**Estudiante:** Mariano Roberto Rac Noguera  
**Carnet:** 202101149  
**Universidad:** Universidad de San Carlos de Guatemala (USAC)  
**Facultad:** Facultad de Ingeniería (FIUSAC)  
**Escuela:** Escuela de Ciencias y Sistemas  
**Curso:** Redes de Computadoras  
**Catedrático:** Ing. Pedro Pablo Hernández  
**Auxiliar:** Adrián Fernández  

---

## 1. Introducción y Objetivos
El presente informe documenta el desarrollo y la implementación de una arquitectura de red jerárquica configurada mediante el software de simulación Cisco Packet Tracer. La topología contempla el diseño estructurado en capas de Núcleo (Core), Distribución y Acceso.

* **Sincronización Centralizada:** Configuración del protocolo VTP (VLAN Trunking Protocol) para simplificar la creación y propagación de VLANs a lo largo de los conmutadores de la topología.
* **Agregación de Enlaces:** Implementación del protocolo PAgP (Port Aggregation Protocol) para conformar enlaces EtherChannel, incrementando el ancho de banda disponible y garantizando tolerancia a fallos entre el switch central y los switches de distribución.
* **Estandarización de Troncales:** Configuración explícita de enlaces troncales IEEE 802.1Q con la VLAN Nativa 99 para el transporte seguro de tráfico no etiquetado y tramas de control.

---

## 2. Parámetros de Red

| Parámetro | Valor Asignado | Descripción Técnica |
| :--- | :--- | :--- |
| **Dominio VTP** | `Smart_4` | Identificador único del dominio de propagación VTP. |
| **Contraseña VTP** | `proyecto12S2026` | Clave MD5 requerida para la autenticación de anuncios VTP. |
| **Modo VTP CORE** | `Server` | Rol del SW-CORE con permisos de lectura/escritura en la base de datos de VLANs. |
| **VLAN Nativa** | `99` | Identificador de VLAN designado para tráfico no etiquetado en troncales. |
| **Protocolo EtherChannel** | `PAgP` | Protocolo propietario de Cisco para negociación dinámica de canales lógicos. |

---

## 3. Configuración Aplicada por Dispositivo

### 3.1. Switch Central (SW-CORE)
Se estableció el modo Server para el protocolo VTP asignando el dominio correspondiente. Posteriormente, se agruparon las interfaces físicas `FastEthernet0/9` y `FastEthernet0/10` bajo la interfaz lógica `Port-channel 1` configurando PAgP en modo `desirable` para iniciar activamente la negociación del enlace.

```cisco
SW-CORE# configure terminal
SW-CORE(config)# vtp mode server
SW-CORE(config)# vtp domain Smart_4
SW-CORE(config)# vtp password proyecto12S2026
SW-CORE(config)# exit

SW-CORE# configure terminal
SW-CORE(config)# interface range fa0/9 - 10
SW-CORE(config-if-range)# switchport mode trunk
SW-CORE(config-if-range)# switchport trunk native vlan 99
SW-CORE(config-if-range)# channel-protocol pagp
SW-CORE(config-if-range)# channel-group 1 mode desirable
SW-CORE(config-if-range)# no shutdown
SW-CORE(config-if-range)# exit

SW-CORE(config)# interface port-channel 1
SW-CORE(config-if)# switchport mode trunk
SW-CORE(config-if)# switchport trunk native vlan 99
SW-CORE(config-if)# no shutdown
SW-CORE(config-if)# exit