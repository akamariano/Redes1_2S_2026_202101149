# Manual Técnico, Práctica 1, Redes de Computadoras 1

Estudiante: Mariano Roberto Rac Noguera
Carné: 202101149
Curso: Redes de Computadoras 1, Segundo Semestre 2026
Empresa del caso: QuetzalDev S.A.

## 1. Inventario de equipos
 
La distribución de PC y laptops se definió según el tipo de trabajo de cada departamento. Se asignaron laptops a las áreas que necesitan movilidad: Dirección General, Diseño e Innovación, Legal y Sala de Capacitación. En los puestos fijos se asignaron PC de escritorio. En total se tienen las 30 PC y 12 laptops indicadas en el enunciado.
 
| Departamento | PC escritorio | Laptops | Servidores | Total hosts | Switch(es) | Categoría de cable |
|---|---|---|---|---|---|---|
| Recursos Humanos | 8 | 0 | 0 | 8 | 1 switch de 12 puertos | Cat6 |
| Sala de Capacitación | 7 | 3 | 0 | 10 | 1 switch de 12 puertos | Cat6 |
| Diseño e Innovación | 4 | 3 | 1 | 8 | 2 switches de 8 puertos (principal y QA) | Cat6 |
| Backend | 6 | 0 | 1 | 7 | 1 switch de 12 puertos | Cat6 |
| Dirección General | 0 | 4 | 0 | 4 | 1 switch de 8 puertos | Cat6 |
| Recepción | 3 | 0 | 1 | 4 | 1 switch de 8 puertos | Cat6 |
| Legal | 2 | 2 | 0 | 4 | 1 switch de 8 puertos | Cat6 |
| Data Center | 0 | 0 | 3 | 3 | 1 switch de 8 puertos | Fibra óptica |
| **Total** | **30** | **12** | **6** | **48** | | |
 
### Equipo activo consolidado

| Elemento | Cantidad | Especificación |
|---|---|---|
| Patch panel | 1 | 48 puertos, Cat6 |
| Switch central (MDF) | 1 | 16 puertos administrable |
| Switches de departamento | 9 | 3 de 12 puertos, 6 de 8 puertos (incluye principal y QA de Diseño e Innovación) |
| Rack | 1 | Rack de pared de 12U |
| UPS | 1 | 650VA a 1000VA |
| ODF (organizador de fibra) | 1 | Instalado en el rack del MDF, para terminar las rutas de fibra óptica del Data Center |
## 2. Ubicación del cuarto de telecomunicaciones (MDF)

Para ubicar el MDF se calculó el centroide ponderado del edificio. En este cálculo, cada departamento tiene una posición (x, y) y un peso igual a su cantidad de hosts. Por eso, los departamentos con más equipos influyen más en la ubicación final. Así se obtiene un punto más útil que el centro geométrico, porque considera la distribución real de los equipos.

La fórmula utilizada fue la siguiente.

x_MDF = Σ(hosts_i × x_i) / Σhosts_i
y_MDF = Σ(hosts_i × y_i) / Σhosts_i

Las coordenadas tienen su origen (0,0) en la esquina inferior izquierda del edificio. El edificio principal mide 26 metros en el eje x y 16 metros en el eje y. El Data Center está fuera de esta área y se conecta por un corredor secundario de 1.5 metros. Por eso, su centro se calculó en x = 30.5.

| Ambiente | x | y | Hosts | Cx (x × hosts) | Cy (y × hosts) |
|---|---|---|---|---|---|
| Recursos Humanos | 2.50 | 4.00 | 8 | 20 | 32 |
| Sala de Capacitación | 7.50 | 4.00 | 10 | 75 | 40 |
| Diseño e Innovación | 17.00 | 4.00 | 8 | 136 | 32 |
| Backend | 23.00 | 4.00 | 7 | 161 | 28 |
| Dirección General | 3.50 | 13.00 | 4 | 14 | 52 |
| Recepción | 13.00 | 13.00 | 4 | 52 | 52 |
| Legal | 22.50 | 13.00 | 4 | 90 | 52 |
| Data Center | 30.50 | 14.00 | 3 | 91.5 | 42 |
| **Total** | | | **48** | **639.5** | **330** |

x_MDF = 639.5 / 48 ≈ 13.32
y_MDF = 330 / 48 ≈ 6.88
URL CÁLCULOS: https://ingenieriausacedu-my.sharepoint.com/:x:/g/personal/3530110610101_ingenieria_usac_edu_gt/IQAJAMek7jhgT6Hr3Tu61K0aAfKd5wrboL2uypH5l7NK5yY?e=tSEymB

El punto resultante (13.32, 6.88) cae dentro del Vestíbulo de Ingreso, cerca del borde que limita con el Hall Central. Esta ubicación es adecuada porque existe un espacio físico para instalar el rack y permite llegar al cableado de las áreas superiores e inferiores. El rack se puede colocar contra una pared lateral sin bloquear la entrada. Se descartó el Hall Central porque es un pasillo de 2 metros de ancho y no tiene un espacio cerrado para instalar el equipo.

Con base en este análisis, el MDF se ubicará en la esquina superior izquierda del Vestíbulo de Ingreso, es decir, del lado hacia donde se abre la puerta de acceso a dicho vestíbulo, quedando pegado al límite con el Hall Central.

## 3. Topología física por departamento

### Data Center

El Data Center concentra los tres servidores principales del edificio, por lo que es el segmento más importante de la red. Una falla en su conexión afectaría a los tres servidores. Entre el switch del Data Center y los servidores se usará una topología en estrella simple, porque hay pocos hosts y una solución más compleja no se justifica.

Para el enlace troncal entre el MDF y el switch del Data Center se evaluará una topología con redundancia, mediante dos rutas de cableado físicamente separadas hacia el mismo switch, dado el nivel de criticidad de este segmento frente al resto del edificio.

Se decidió usar un enlace troncal redundante para el Data Center, con dos rutas de cableado separadas hacia el mismo switch. En Backend y Recepción, una falla afecta principalmente a un solo departamento. En cambio, el Data Center contiene servidores que utiliza toda la empresa. El costo de agregar una segunda ruta es menor que el riesgo de perder el acceso a todos los servidores por una falla en un cable o puerto.
### Backend

Backend tiene 6 estaciones de desarrollo y 1 servidor, para un total de 7 hosts. Como la mayoría son estaciones de trabajo, el segmento es menos crítico que el Data Center. Se usará una topología en estrella simple porque un solo switch puede atender todos los hosts.

El enlace entre el MDF y Backend será simple y no tendrá redundancia. Solo uno de sus siete hosts es un servidor y las demás estaciones pueden reemplazarse. En este caso se prioriza el costo sobre la tolerancia a fallos.
### Recepción
 
Recepción tiene 3 estaciones de trabajo y 1 servidor, para un total de 4 hosts. Como solo uno es servidor, se aplica el mismo criterio que en Backend. Se usará una topología en estrella simple porque hay pocos hosts.
 
El enlace entre el MDF y Recepción será simple y no tendrá redundancia. El impacto de una falla temporal sería bajo, por lo que se prioriza el costo.
 
### Recursos Humanos
 
El Departamento de Recursos Humanos aloja 8 estaciones de trabajo, sin servidores. Internamente, entre el switch de Recursos Humanos y sus 8 hosts, se utilizará una topología en estrella simple, ya que la cantidad de hosts es manejable por un solo switch y no hay equipos críticos que justifiquen una arquitectura más compleja.
 
Para el enlace troncal entre el MDF y el switch de Recursos Humanos se utilizará un enlace simple sin redundancia, dado que el segmento no aloja servidores y su criticidad es baja frente al resto del edificio.
 
### Legal
 
El Departamento Legal aloja 4 estaciones de trabajo, sin servidores. Internamente, entre el switch de Legal y sus 4 hosts, se utilizará una topología en estrella simple, dado el bajo número de hosts y la ausencia de equipos críticos.
 
Para el enlace troncal entre el MDF y el switch de Legal se utilizará un enlace simple sin redundancia, siguiendo el mismo criterio de bajo costo aplicado en los segmentos sin servidores.
 
### Sala de Capacitación
 
La Sala de Capacitación aloja 10 estaciones de trabajo, sin servidores, la mayor cantidad de hosts entre los departamentos sin equipo crítico. Internamente, entre el switch de Sala de Capacitación y sus 10 hosts, se utilizará una topología en estrella simple, ya que un switch de capacidad adecuada (24 puertos o similar) maneja sin problema esa cantidad de hosts sin necesidad de dividir el segmento en más de un switch.
 
Para el enlace troncal entre el MDF y el switch de Sala de Capacitación se utilizará un enlace simple sin redundancia, dado que el segmento no aloja servidores y su criticidad operativa es baja.
 
### Dirección General
 
Dirección General aloja 4 estaciones de trabajo, sin servidores. Internamente, entre el switch de Dirección General y sus 4 hosts, se utilizará una topología en estrella simple, dado el bajo número de hosts.
 
Para el enlace troncal entre el MDF y el switch de Dirección General se utilizará un enlace simple sin redundancia, siguiendo el mismo criterio de costo aplicado en los demás segmentos sin equipo crítico.
 
### Diseño e Innovación
 
El Departamento de Diseño e Innovación agrupa tres áreas de trabajo (Diseño UI/UX, Data Analytics y Laboratorio de Pruebas QA), con 7 estaciones de trabajo y 1 servidor, 8 hosts en total. Se propone una topología en estrella extendida a nivel interno, dividiendo el segmento en dos switches en cascada. El primer switch conecta 5 estaciones de Diseño UI/UX y Data Analytics junto con el servidor del departamento, dado que este servidor alimenta de datos y assets a ambas áreas de forma constante. El segundo switch, más pequeño, conecta las 2 estaciones restantes del Laboratorio de Pruebas QA, un área que consume esos recursos de forma puntual y no los aloja, por lo que no requiere compartir el mismo switch que el servidor.
 
Esta separación se justifica porque el Laboratorio de QA suele generar tráfico de pruebas distinto al del resto del área, y aislarlo en su propio switch facilita el diagnóstico de fallas sin afectar a Diseño y Data Analytics. El incremento de costo es bajo, ya que el switch dedicado a QA puede ser un modelo pequeño de 5 u 8 puertos, suficiente para sus 2 hosts.
 
Para el enlace troncal entre el MDF y el switch principal de Diseño e Innovación se utilizará un enlace simple sin redundancia, dado que solo uno de los 8 hosts del segmento es un servidor.



## 4. Medio de transmisión
 
El estándar TIA/EIA-568 limita cualquier segmento de cobre UTP a 100 metros totales (90 metros de cableado fijo más 10 metros de patch cords en ambos extremos). Dado que el edificio mide 26 por 16 metros, ninguna de las distancias calculadas en la sección 5 se acerca a ese límite, por lo que la selección de categoría de cable se basó en el ancho de banda requerido y en la criticidad de cada segmento, no en la distancia.
 
Se seleccionó Cat6 para todos los segmentos de cableado horizontal y troncal de los siete departamentos sin criticidad especial (RRHH, Sala de Capacitación, Diseño e Innovación, Backend, Dirección General, Recepción y Legal). Cat6 soporta 1 Gbps garantizado en los 100 metros completos, suficiente para estaciones de trabajo y equipo de oficina estándar, y representa un costo menor frente a categorías superiores sin sacrificar el desempeño que estos segmentos requieren.
 
Para el Data Center se seleccionó fibra óptica, tanto para el cableado horizontal hacia los tres servidores como para las dos rutas troncales redundantes hacia el switch central. La decisión se sustenta en tres factores. Primero, es el segmento que concentra el tráfico de los servidores principales hacia el resto del edificio, por lo que requiere mayor ancho de banda que cualquier otro segmento. Segundo, ya se había decidido una topología troncal redundante para este segmento por su criticidad, y la fibra es el medio estándar de la industria para backbones críticos con redundancia, al ofrecer mayor inmunidad a interferencia electromagnética que el cobre. Tercero, el sobrecosto de usar fibra se limita únicamente a este segmento, que es corto en comparación con el resto del cableado del edificio, por lo que el impacto en el presupuesto total es acotado frente al beneficio de contar con el segmento más crítico del edificio en el medio de mayor capacidad y confiabilidad disponible.

## 5. Distancias de cableado y cálculo de bobinas
 
### 5.1 Cableado troncal (MDF hacia switch de cada departamento)
 
Las distancias se estimaron con distancia Manhattan (suma de la diferencia en x más la diferencia en y entre el MDF y el centro de cada departamento), sobre las coordenadas del plano con origen en la esquina inferior izquierda del edificio. Sobre esa distancia se aplicó un margen del 20% para cubrir subidas y bajadas hacia la canalización en el techo y el service loop en ambos extremos.
 
El MDF se tomó en el punto redondeado (10, 8), aproximación del centroide ponderado calculado en la sección 2.
 
| Ambiente | x | y | Dx | Dy | Suma (m) | Distancia con margen 20% (m) |
|---|---|---|---|---|---|---|
| Recursos Humanos | 2.5 | 4 | 7.5 | 4 | 11.5 | 13.8 |
| Sala de Capacitación | 7.5 | 4 | 2.5 | 4 | 6.5 | 7.8 |
| Diseño e Innovación | 17 | 4 | 7 | 4 | 11 | 13.2 |
| Backend | 23 | 4 | 13 | 4 | 17 | 20.4 |
| Dirección General | 3.5 | 13 | 6.5 | 5 | 11.5 | 13.8 |
| Recepción | 13 | 13 | 3 | 5 | 8 | 9.6 |
| Legal | 22.5 | 13 | 12.5 | 5 | 17.5 | 21 |
| Data Center | 30.5 | 14 | 20.5 | 6 | 26.5 | 31.8 |
 
Dado que el enlace troncal hacia el Data Center es redundante (dos rutas físicas separadas), su distancia se contabiliza dos veces para efectos del material a comprar.
 
Suma de los siete departamentos con ruta simple: 13.8 + 7.8 + 13.2 + 20.4 + 13.8 + 9.6 + 21 = 99.6 metros.
Ruta doble del Data Center: 31.8 × 2 = 63.6 metros.
Total de cableado troncal requerido: 99.6 + 63.6 = 163.2 metros.
 
Con bobinas estándar de 305 metros, 163.2 / 305 ≈ 0.53, por lo que se requiere 1 bobina completa para cubrir el cableado troncal del edificio, quedando un remanente aproximado de 141.8 metros disponible como reserva.
 
### 5.2 Cableado horizontal (switch de departamento hacia cada host)
 
Las distancias se estimaron ubicando el switch de cada departamento en una esquina de la sala, cercana al lado por donde sale el cableado troncal hacia el MDF, y midiendo la distancia Manhattan hacia el host más alejado dentro de esa misma sala, con un margen del 20% adicional.
 
| Ambiente | Ancho (m) | Largo (m) | Ubicación del switch | Distancia máxima con margen 20% (m) |
|---|---|---|---|---|
| RRHH | 5 | 8 | Esquina superior derecha | 15.6 |
| Sala de Capacitación | 5 | 8 | Esquina superior derecha | 15.6 |
| Diseño e Innovación | 6 | 8 | Esquina superior derecha | 16.8 |
| Backend | 6 | 8 | Esquina superior izquierda | 16.8 |
| Dirección General | 7 | 6 | Esquina superior derecha | 15.6 |
| Recepción | 12 | 6 | Esquina superior derecha | 21.6 |
| Legal | 7 | 6 | Esquina superior derecha | 15.6 |
| Data Center | 6 | 4 | Esquina superior derecha | 12.0 |
 
Para estimar el material necesario se multiplicó la distancia máxima de cada departamento por su cantidad de hosts, asumiendo de forma conservadora que cada host requiere un tendido similar al del punto más alejado de la sala. Este criterio sobreestima levemente el material real, pero evita quedarse corto en la compra de cable.
 
| Ambiente | Distancia máxima (m) | Hosts | Metros estimados |
|---|---|---|---|
| RRHH | 15.6 | 8 | 124.8 |
| Sala de Capacitación | 15.6 | 10 | 156.0 |
| Diseño e Innovación | 16.8 | 8 | 134.4 |
| Backend | 16.8 | 7 | 117.6 |
| Dirección General | 15.6 | 4 | 62.4 |
| Recepción | 21.6 | 4 | 86.4 |
| Legal | 15.6 | 4 | 62.4 |
| Data Center | 12.0 | 3 | 36.0 |
| **Total** | | | **780.0** |
 
Con bobinas estándar de 305 metros, 780 / 305 ≈ 2.56, por lo que se requieren 3 bobinas completas para el cableado horizontal, con un remanente aproximado de 135 metros.
 
Sumando el cableado troncal (1 bobina, sección 5.1) con el cableado horizontal (3 bobinas), el total de bobinas de cable UTP requeridas para todo el edificio es de 4 bobinas.

## 6. Dimensionamiento de patch panel y switch
 
### 6.1 Patch panel
 
El patch panel del MDF se dimensiona según el total de puntos de red del edificio, sumando los hosts de los ocho departamentos: 8 (RRHH) + 10 (Sala de Capacitación) + 8 (Diseño e Innovación) + 7 (Backend) + 4 (Dirección General) + 4 (Recepción) + 4 (Legal) + 3 (Data Center) = 48 puntos. Se seleccionó un patch panel estándar de 48 puertos, que cubre exactamente el total sin desperdicio de espacio en el rack.
 
### 6.2 Switch central (MDF)
 
El switch central debe tener un puerto por cada conexión troncal que llega al MDF. Siete de los ocho departamentos (RRHH, Sala de Capacitación, Diseño e Innovación, Backend, Dirección General, Recepción y Legal) ocupan un solo puerto cada uno, ya que el switch secundario de QA dentro de Diseño e Innovación se conecta a su propio switch principal departamental y no directo al MDF. El Data Center ocupa 2 puertos, por su enlace troncal redundante de dos rutas físicas separadas. El total de conexiones en el switch central es 7 + 2 = 9. Se seleccionó un switch central administrable de 16 puertos, dejando margen para escalabilidad futura.
 
### 6.3 Switches de departamento
 
Cada switch de departamento requiere un puerto por host conectado más los puertos de enlace troncal correspondientes (1 para la mayoría de departamentos, 2 para el Data Center por su redundancia, y 1 adicional en el switch principal de Diseño e Innovación para el enlace hacia su switch secundario de QA).
 
| Ambiente | Hosts | Puertos requeridos | Switch seleccionado |
|---|---|---|---|
| RRHH | 8 | 9 (8 hosts + 1 troncal) | 12 puertos |
| Sala de Capacitación | 10 | 11 (10 hosts + 1 troncal) | 12 puertos |
| Diseño e Innovación (switch principal) | 6 | 8 (6 hosts + 1 troncal MDF + 1 enlace a switch QA) | 8 puertos |
| Diseño e Innovación (switch QA) | 2 | 3 (2 hosts + 1 enlace al switch principal) | 8 puertos |
| Backend | 7 | 8 (7 hosts + 1 troncal) | 12 puertos |
| Dirección General | 4 | 5 (4 hosts + 1 troncal) | 8 puertos |
| Recepción | 4 | 5 (4 hosts + 1 troncal) | 8 puertos |
| Legal | 4 | 5 (4 hosts + 1 troncal) | 8 puertos |
| Data Center | 3 | 5 (3 hosts + 2 troncal redundante) | 8 puertos |
 
En RRHH, Sala de Capacitación y Backend se seleccionó un switch de 12 puertos en vez del mínimo ajustado, dejando margen de crecimiento sobre el cálculo exacto.

## 7. Canalización
 
Se seleccionó escalerilla abierta (tipo charola) sobre cielo falso para todo el edificio, tanto para el cableado troncal como para el horizontal. Esta opción es más económica que la escalerilla cerrada o la tubería, y facilita la inspección y ampliación futura del cableado. Se descartó la escalerilla cerrada porque el edificio no presenta ambientes industriales ni fuentes significativas de interferencia electromagnética que justifiquen el costo adicional de un ducto cerrado.
 
## 8. Rack o gabinete para el MDF
 
Se seleccionó un rack de pared de 12U para el MDF. El dimensionamiento se calculó sumando el espacio real que ocupa cada componente: el switch central administrable de 16 puertos ocupa 1U, el patch panel de 48 puertos ocupa 2U, se agregan organizadores de cable de 1U antes y después de cada componente activo (2U en total) para evitar enredos y mantener la ventilación, y 1U adicional para el PDU (regleta de energía del rack). Esto suma 6U de uso real, dejando los 6U restantes como margen de crecimiento, siguiendo la práctica de dejar entre 40% y 50% de espacio libre en un rack nuevo, ya que ampliar la capacidad después de instalado y cableado resulta mucho más costoso que dimensionarlo con margen desde el inicio.
 
## 9. Respaldo de energía (UPS)
 
Se seleccionó un UPS de entre 650VA y 1000VA para el MDF. El consumo a respaldar corresponde principalmente al switch central (aproximadamente 30 a 60 watts para un modelo administrable de 16 puertos), por lo que ese rango de capacidad resulta holgado y ofrece varios minutos de autonomía, suficiente para un apagado controlado del equipo o para que el generador de respaldo del edificio, si existe, tome la carga.

## 10. Estándares T568A/T568B, straight-through y crossover
 
Todos los enlaces del edificio, tanto el cableado horizontal (host hacia switch de departamento) como el cableado troncal (switch de departamento hacia switch central), se implementan con cables straight-through, es decir, con la misma disposición de pines en ambos extremos del cable. Esta decisión se sustenta en que, aunque tradicionalmente los enlaces switch-switch requerían cables crossover, el equipo de red actual cuenta con la función auto-MDI-X, que detecta automáticamente la polaridad del enlace y la ajusta sin necesidad de un cable crossover físico. El cable crossover se documenta únicamente como concepto de respaldo, para el caso hipotético de que se utilice equipo sin auto-MDI-X.
 
Se seleccionó el estándar T568B para la asignación de colores en ambos extremos de todos los cables del edificio, por ser el más utilizado comercialmente y el más común en instalaciones actuales. Se mantiene consistencia total, usando T568B en ambos extremos de cada cable, ya que al no requerirse ningún enlace crossover en este diseño, no es necesario mezclar T568A con T568B en ningún cable.
 
## 11. Disposición de pines
 
La siguiente tabla muestra la disposición de pines del estándar T568B, aplicada en ambos extremos de todos los cables straight-through del edificio.
 
| Pin | Color |
|---|---|
| 1 | Blanco/Naranja |
| 2 | Naranja |
| 3 | Blanco/Verde |
| 4 | Azul |
| 5 | Blanco/Azul |
| 6 | Verde |
| 7 | Blanco/Café |
| 8 | Café |

Como evidencia de comprensión de la disposición de pines para el caso de un cable crossover, aunque el diseño de este edificio no requiere ninguno, se documenta a continuación el pinout correspondiente, con T568B en un extremo y T568A en el otro, que es la combinación que definiría un cable crossover si en algún momento se necesitara conectar dos switches sin auto-MDI-X.

| Pin | Extremo A (T568B) | Extremo B (T568A) |
|---|---|---|
| 1 | Blanco/Naranja | Blanco/Verde |
| 2 | Naranja | Verde |
| 3 | Blanco/Verde | Blanco/Naranja |
| 4 | Azul | Azul |
| 5 | Blanco/Azul | Blanco/Azul |
| 6 | Verde | Naranja |
| 7 | Blanco/Café | Blanco/Café |
| 8 | Café | Café |
## 12. Tipo de toma de red y etiquetado de cableado
 
Todos los puntos de red del edificio se implementan con toma unitaria (un solo puerto por toma), ya que el diseño asigna una corrida de cable dedicada e independiente a cada host, servidor o punto de conexión, sin compartir una misma toma entre dos o más dispositivos. No se requieren tomas dobles, triples ni de N puertos en ningún ambiente, dado que ningún punto de la red concentra más de un dispositivo físico en una sola salida.
 
Se utilizará el siguiente formato de etiquetado para todos los cables del edificio, tal como lo exige el enunciado de la práctica.
 
Para el cableado horizontal se usará el formato `[Área/Departamento]-PR[Número de Punto de Red]`. Ejemplos: RRHH-PR01 a RRHH-PR08, SalaCap-PR01 a SalaCap-PR10, DIN-PR01 a DIN-PR08, Backend-PR01 a Backend-PR07, DirGeneral-PR01 a DirGeneral-PR04, Recepcion-PR01 a Recepcion-PR04, Legal-PR01 a Legal-PR04 y DataCenter-PR01 a DataCenter-PR03. Los dos puntos del switch de QA se incluyen en la numeración de Diseño e Innovación.
 
Para el cableado troncal se usará el formato `MDF-[Área/Departamento]`. Ejemplos: MDF-RRHH, MDF-SalaCap, MDF-DIN, MDF-Backend, MDF-DirGeneral, MDF-Recepcion y MDF-Legal. Las rutas del Data Center se identificarán como MDF-DataCenter-A y MDF-DataCenter-B. El enlace entre los switches de Diseño e Innovación se etiquetará como DIN-DIN-QA.
 
### Comparación con el estándar TIA/EIA-606

El estándar ANSI/TIA-606 define cómo identificar y registrar los elementos de una red de telecomunicaciones. A diferencia del TIA/EIA-568, que se enfoca en el desempeño del cableado, el 606 se enfoca en la identificación, el etiquetado y el registro de la infraestructura.

En esta práctica solo se etiquetan los cables. El estándar 606 también pide identificadores únicos para los puertos del patch panel, los cuartos de telecomunicaciones, las canalizaciones y el sistema de puesta a tierra. Además, recomienda usar colores como apoyo visual y mantener un registro formal de todos los elementos. En un entorno real se usaría el estándar completo, porque facilita encontrar equipos y resolver fallas cuando participan varios técnicos y la red cambia con el tiempo.


## 13. Flujo de conexión end-to-end
 
El enunciado exige documentar el flujo completo de al menos un dispositivo final. A continuación se documentan dos casos representativos, uno de un segmento estándar y otro del segmento crítico con redundancia, para ilustrar ambos escenarios de la red.
 
### Caso 1, estación de trabajo en Backend
 
1. La estación de trabajo transmite la señal desde su tarjeta de red (NIC).
2. La señal viaja por el cable de cableado horizontal Cat6, etiquetado como Backend-PR01, tendido por escalerilla abierta desde la estación hasta el switch del departamento.
3. La señal llega a un puerto del switch de Backend (switch de 12 puertos).
4. El switch de Backend reenvía la señal por el cable troncal Cat6, etiquetado como MDF-Backend, también por escalerilla abierta, hacia el switch central.
5. La señal llega a un puerto del switch central de 16 puertos, ubicado en el rack de 12U del MDF.
6. Desde el switch central, la señal pasa al patch panel de 48 puertos del MDF, que la deja disponible para su salida hacia el resto de la red del edificio o hacia el proveedor de servicio de internet, según la topología de conexión externa definida por la empresa.
### Caso 2, servidor en Data Center (segmento con redundancia)
 
1. El servidor transmite la señal desde su tarjeta de red (NIC).
2. La señal viaja por cable de fibra óptica horizontal, etiquetado como DataCenter-PR01, hasta el switch del Data Center.
3. La señal llega a un puerto del switch del Data Center (switch de 8 puertos).
4. El switch del Data Center reenvía la señal simultáneamente por dos rutas de fibra óptica troncal físicamente separadas, etiquetadas como MDF-DataCenter-A y MDF-DataCenter-B, cada una por su propia canalización, hacia el switch central.
5. Ambas rutas llegan a puertos independientes del switch central de 16 puertos en el MDF. Si una de las dos rutas falla (corte de cable, falla de puerto), la segunda ruta mantiene la conectividad del Data Center sin interrupción del servicio.
6. Desde el switch central, la señal pasa al patch panel de 48 puertos del MDF, y de ahí hacia el resto de la red del edificio o hacia el proveedor de servicio de internet.
La diferencia clave entre ambos casos es que el segmento de Backend depende de una única ruta física en cada tramo (horizontal y troncal), mientras que el segmento del Data Center cuenta con redundancia física completa en el tramo troncal, coherente con la decisión tomada en la sección 3 de dar tratamiento especial de alta disponibilidad al segmento que aloja los servidores principales del edificio.
## 14. Presupuesto estimado
 
A continuación se presenta el presupuesto estimado de materiales, con precios referenciales del mercado local de Guatemala en quetzales, y sus fuentes de referencia.
 
| Rubro | Cantidad | Precio unitario | Precio total | Referencia |
|---|---|---|---|---|
| Cable UTP Cat6, bobina de 305m (segmento horizontal y troncal) | 3 bobinas | Q 704.00 | Q 2,112.00 | Kemik Guatemala |
| Fibra óptica multimodo OM3, por metro (segmento Data Center) | ~30 m | Q 16.28 | Q 488.40 | Chipcom Guatemala |
| Patch panel Cat6 de 48 puertos | 1 ud. | Q 622.00 | Q 622.00 | Kemik Guatemala |
| Switch central Gigabit de 16 puertos | 1 ud. | Q 615.00 | Q 615.00 | Mivoot Guatemala |
| Switches de departamento, 12 puertos Gigabit | 3 uds. | Q 535.00 | Q 1,605.00 | Mivoot Guatemala |
| Switches de departamento, 8 puertos Gigabit (4 de departamento + 2 de Diseño e Innovación) | 6 uds. | Q 295.00 | Q 1,770.00 | Mivoot Guatemala |
| Rack de pared de 12U | 1 ud. | Q 1,905.60 | Q 1,905.60 | Conectividad Guatemala |
| UPS de 650VA a 1000VA | 1 ud. | Q 567.00 | Q 567.00 | Walmart Guatemala |
| Conectores RJ45 Cat6, bolsa de 100 unidades | 1 bolsa | Q 265.00 | Q 265.00 | Llego Store Guatemala |
| Escalerilla abierta / charola malla, tramo de 3m | ~5 tramos (15 m) | Q 349.58 | Q 1,747.90 | Chipcom Guatemala |
 
Costo total aproximado en materiales: Q 11,697.90 (impuestos incluidos en la mayoría de los distribuidores locales consultados).
 
Consideraciones adicionales sobre el presupuesto. Para la fibra óptica del Data Center se presupuestaron 30 metros de fibra multimodo OM3, cantidad que cubre el tramo corto de este segmento; adicionalmente se requerirán conectores LC o SC y su respectivo patch cord o acoplador en un patch panel de fibra dedicado, no incluidos en el desglose anterior. Para la escalerilla se estimaron 5 tramos de 3 metros (15 metros en total) de charola tipo malla de 54 por 100 milímetros, cantidad que representa la ruta principal compartida por donde corren agrupados los cables hacia cada departamento, y no la suma de las longitudes individuales de cada cable, ya que múltiples cables viajan juntos dentro de la misma canalización; esta cantidad deberá confirmarse contra el recorrido real una vez trazado el diagrama final. Una bolsa de 100 conectores RJ45 cubre los extremos necesarios para el parcheo de las redes horizontal y troncal de cobre del edificio.