# Manual Técnico, Práctica 1, Redes de Computadoras 1

Estudiante: Mariano Roberto Rac Noguera
Carné: 202101149
Curso: Redes de Computadoras 1, Segundo Semestre 2026
Empresa del caso: QuetzalDev S.A.

## 1. Inventario de equipos

Pendiente. Se detallará una vez definidas las topologías por departamento y el dimensionamiento del patch panel y del switch central.

## 2. Ubicación del cuarto de telecomunicaciones (MDF)

Para ubicar el MDF se calculó el centroide ponderado del edificio, tomando como peso la cantidad de hosts de cada departamento. Este método corresponde al mismo principio del centro de masa en física, donde cada departamento actúa como un punto con una posición (x, y) sobre el plano y un peso equivalente a su cantidad de equipos. El departamento con más hosts atrae el punto resultante con más fuerza que uno con pocos hosts, de manera que el MDF queda ubicado en el punto que minimiza la distancia ponderada promedio hacia todos los puntos de red del edificio, en lugar de simplemente ubicarse en el centro geométrico del edificio.

La fórmula utilizada fue la siguiente.

x_MDF = Σ(hosts_i × x_i) / Σhosts_i
y_MDF = Σ(hosts_i × y_i) / Σhosts_i

Las coordenadas se tomaron con origen (0,0) en la esquina inferior izquierda del edificio, con el eje x corriendo de 0 a 26 metros y el eje y de 0 a 16 metros para el edificio principal. Para el Data Center, ubicado fuera de la franja principal y conectado mediante el Corredor Secundario de 1.5 metros de ancho, se estimó su centro en x igual a 26 más 1.5 más la mitad de su ancho de 6 metros, dando como resultado x igual a 30.5.

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

El punto resultante (13.32, 6.88) cae dentro del área del Vestíbulo de Ingreso, muy cerca de su borde superior con el Hall Central. Esta zona resulta viable como cuarto de telecomunicaciones porque es un espacio físico real y no un punto en el aire, permite el acceso directo tanto al cableado hacia los departamentos de la fila inferior como hacia los de la fila superior a través del Hall Central, y no interrumpe la circulación principal de ingreso si se ubica contra una pared lateral. Se descartó el Hall Central como ubicación alternativa, a pesar de estar geométricamente aún más cerca del centroide, porque es un pasillo de 2 metros de ancho sin espacio cerrado para instalar un rack o gabinete.

Con base en este análisis, el MDF se ubicará en la esquina superior izquierda del Vestíbulo de Ingreso, es decir, del lado hacia donde se abre la puerta de acceso a dicho vestíbulo, quedando pegado al límite con el Hall Central.

## 3. Topología física por departamento

### Data Center

El Data Center concentra los tres servidores principales del edificio, por lo que representa el segmento de mayor criticidad de toda la red, dado que una falla en su conexión afectaría de forma simultánea a los tres servidores centrales. Internamente, entre el switch del Data Center y sus tres servidores, se utilizará una topología en estrella simple, ya que con tan pocos hosts cualquier topología más compleja no se justifica en términos de costo.

Para el enlace troncal entre el MDF y el switch del Data Center se evaluará una topología con redundancia, mediante dos rutas de cableado físicamente separadas hacia el mismo switch, dado el nivel de criticidad de este segmento frente al resto del edificio.

Se decidió un enlace troncal redundante para el Data Center, mediante dos rutas de cableado físicamente separadas hacia el mismo switch. A diferencia de Backend y Recepción, donde el servidor departamental afecta únicamente a ese departamento si falla, el Data Center aloja los tres servidores principales del edificio, es decir, infraestructura compartida por toda la empresa. El costo adicional de una segunda ruta de cable es marginal frente al riesgo de que una falla en un solo cable o puerto deje sin acceso a los servidores centrales a la totalidad de QuetzalDev S.A., por lo que la relación costo-beneficio favorece la redundancia en este segmento específico.
### Backend

El Departamento de Backend aloja 6 estaciones de desarrollo y 1 servidor, 7 hosts en total. A diferencia del Data Center, donde todos los hosts son servidores, aquí la mayoría son estaciones de trabajo normales, por lo que la criticidad del segmento completo es menor. Internamente, entre el switch de Backend y sus 7 hosts, se utilizará una topología en estrella simple, ya que la cantidad de hosts no justifica dividir el switch en más de un grupo.

Para el enlace troncal entre el MDF y el switch de Backend se utilizará un enlace simple sin redundancia, dado que solo uno de los 7 hosts del segmento es un servidor, y el resto son estaciones reemplazables sin un impacto crítico en la operación del edificio si llegara a fallar temporalmente la conexión. Esta decisión prioriza el costo sobre la tolerancia a fallos, a diferencia del Data Center donde los tres hosts conectados son servidores.
### Recepción
 
El Departamento de Recepción aloja 3 estaciones de trabajo y 1 servidor, 4 hosts en total. Al igual que Backend, la mayoría de los hosts son estaciones reemplazables y solo uno es un servidor, por lo que se aplica el mismo criterio. Internamente, entre el switch de Recepción y sus 4 hosts, se utilizará una topología en estrella simple, dado el bajo número de hosts.
 
Para el enlace troncal entre el MDF y el switch de Recepción se utilizará un enlace simple sin redundancia, siguiendo la misma lógica aplicada en Backend, priorizando el costo dado que el impacto de una falla temporal en este segmento es bajo.
 
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

Pendiente.

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

## 6. Dimensionamiento de patch panel y switch

Pendiente.

## 7. Canalización

Pendiente.

## 8. Rack o gabinete para el MDF

Pendiente.

## 9. Respaldo de energía (UPS)

Pendiente.

## 10. Estándares T568A/T568B, straight-through y crossover

Pendiente.

## 11. Disposición de pines

Pendiente.

## 12. Etiquetado de cableado y comparación con TIA/EIA-606

Pendiente.

## 13. Flujo de conexión end-to-end

Pendiente.

## 14. Presupuesto estimado
