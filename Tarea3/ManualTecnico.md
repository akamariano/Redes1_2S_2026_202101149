# Manual Técnico - Tarea #3

**Estudiante:** Mariano Roberto Rac Noguera  
**Carné:** 202101149  
**Curso:** Redes de Computadoras 1, Segundo Semestre 2026  

---

## 1. Introducción y Descripción de la Topología

El presente manual técnico documenta el diseño, la configuración y la verificación de una topología de red segmentada mediante **VLANs** y gestionada centralizadamente mediante el protocolo **VTP (VLAN Trunking Protocol)** en Cisco Packet Tracer.

La red está conformada por un switch central (`Switch0`) configurado en modo servidor VTP y tres switches de acceso distribuidos por departamento:
* **`ADMIN`:** Configurado como cliente VTP. Almacena puertos asignados a la **VLAN 10** (`192.168.10.0/24`).
* **`MERCA`:** Configurado como cliente VTP. Almacena puertos asignados a la **VLAN 20** (`192.168.20.0/24`).
* **`VENTAS`:** Configurado en modo transparente VTP. Administra localmente la **VLAN 30** (`192.168.30.0/24`).

Los enlaces entre el switch central y los switches de acceso se configuraron en modo troncal (*trunk*) para permitir la propagación de las tramas etiquetadas IEEE 802.1Q de las distintas VLANs.

---

## 2. Scripts de Configuración CLI

A continuación se presentan las configuraciones aplicadas en la interfaz de línea de comandos (CLI) de cada switch.

### 2.1 Switch Central (`Switch0` - VTP Server)
```text
enable
configure terminal
hostname Switch0

! Configuración VTP Servidor
vtp domain FIUSAC
vtp mode server

! Creación de VLANs en el Servidor
vlan 10
 name ADMIN
vlan 20
 name MERCA
vlan 30
 name VENTAS
exit

! Configuración de Enlaces Troncales GigabitEthernet (hacia ADMIN y MERCA)
interface range GigabitEthernet 0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan all
 no shutdown
exit

! Configuración de Enlace Troncal FastEthernet (hacia VENTAS)
interface FastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan all
 no shutdown
exit

end
write memory