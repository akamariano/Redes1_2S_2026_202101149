# Redes de Computadoras 1

Mariano Roberto Rac Noguera · carné 202101149 · Universidad de San Carlos de Guatemala, segundo semestre 2026

Prácticas, tarea y proyecto del curso, diseñados y configurados en Cisco Packet Tracer: desde el cableado estructurado de un edificio hasta un campus jerárquico con VTP, EtherChannel y Rapid PVST+. El proyecto final de enrutamiento (BanTech GT: OSPF, EIGRP, RIPv2, HSRP y redistribución) está en su propio repositorio, [Proyecto3_Redes1_2S_2026_202101149](https://github.com/akamariano/Proyecto3_Redes1_2S_2026_202101149).

| Carpeta | Red | Lo que demuestra |
|---|---|---|
| [Practica1](Practica1/) | Cableado estructurado de un edificio de oficinas de 48 hosts | MDF ubicado por centroide ponderado, distancias Manhattan con holgura del 20 %, Cat6 en los departamentos y fibra OM3 en el Data Center, dimensionamiento de switches y presupuesto según TIA/EIA-568 |
| [Tarea3](Tarea3/) | 4 switches por departamento | VLANs, VTP en modo servidor, cliente y transparente, y troncales 802.1Q |
| [Proyecto1](Proyecto1/) | SmartCity Tech Park: campus jerárquico (núcleo, distribución y acceso) | Dominio VTP con contraseña, 5 VLANs propagadas desde el núcleo, VLAN nativa 99, EtherChannel PAgP, Rapid PVST+, malla completa en I+D, banner MOTD y CDP deshabilitado hacia la granja de servidores |
| [Practica2](Practica2/) | Ciudad Cayalá: red de servicios comunes con 15 switches | Topología híbrida con triángulo redundante en el núcleo, VLANs por zona, EtherChannel, root bridges planificados con Rapid PVST+, VLAN blackhole para puertos sin uso |

Cada carpeta trae su manual técnico, las configuraciones de CLI en archivos `.conf` o `.txt` y el archivo `.pkt`.

## Cómo probarlo

1. Descarga Cisco Packet Tracer 8.2.2 o superior (gratis con una cuenta de [Cisco Networking Academy](https://www.netacad.com/resources/lab-downloads)).
2. Clona el repositorio o descarga el `.pkt` de la carpeta que quieras revisar y ábrelo.
3. Entra a la CLI de cualquier switch (pestaña *CLI*, luego `enable`) y verifica:

| Para revisar | Comando |
|---|---|
| VLANs y puertos asignados | `show vlan brief` |
| Dominio, modo y revisión de VTP | `show vtp status` |
| Troncales y VLANs permitidas | `show interfaces trunk` |
| EtherChannel | `show etherchannel summary` |
| Root bridge y estado de cada puerto en Spanning Tree | `show spanning-tree` |

4. Desde una PC (*Desktop*, luego *Command Prompt*) haz `ping` a otra PC de la misma VLAN: responde. A una de otra VLAN no responde, porque estas redes son de capa 2 y no hay enrutamiento entre VLANs. Las pruebas esperadas, con capturas, están en el manual de cada carpeta.

Para reconstruir una red desde cero en un archivo nuevo, pega en la CLI de cada switch los archivos `.conf` (Practica2), `configs/*.txt` (Proyecto1) o `scripts.txt` (Tarea3), en el orden que indica cada manual.
