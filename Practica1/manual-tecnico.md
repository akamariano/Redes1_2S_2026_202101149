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

El punto resultante (13.32, 6.88) cae dentro del área del Vestíbulo de Ingreso, muy cerca de su borde superior con el Hall Central. Esta zona resulta viable como cuarto de telecomunicaciones porque es un espacio físico real y no un punto en el aire, permite el acceso directo tanto al cableado hacia los departamentos de la fila inferior como hacia los de la fila superior a través del Hall Central, y no interrumpe la circulación principal de ingreso si se ubica contra una pared lateral. Se descartó el Hall Central como ubicación alternativa, a pesar de estar geométricamente aún más cerca del centroide, porque es un pasillo de 2 metros de ancho sin espacio cerrado para instalar un rack o gabinete.

Con base en este análisis, el MDF se ubicará en la esquina superior izquierda del Vestíbulo de Ingreso, es decir, del lado hacia donde se abre la puerta de acceso a dicho vestíbulo, quedando pegado al límite con el Hall Central.

## 3. Topología física por departamento

### Data Center

El Data Center concentra los tres servidores principales del edificio, por lo que representa el segmento de mayor criticidad de toda la red, dado que una falla en su conexión afectaría de forma simultánea a los tres servidores centrales. Internamente, entre el switch del Data Center y sus tres servidores, se utilizará una topología en estrella simple, ya que con tan pocos hosts cualquier topología más compleja no se justifica en términos de costo.

Para el enlace troncal entre el MDF y el switch del Data Center se evaluará una topología con redundancia, mediante dos rutas de cableado físicamente separadas hacia el mismo switch, dado el nivel de criticidad de este segmento frente al resto del edificio.

(Pendiente: justificación final de la topología troncal del Data Center, y definición del resto de departamentos: Recursos Humanos, Sala de Capacitación, Diseño e Innovación, Backend, Dirección General, Recepción y Legal.)

## 4. Medio de transmisión

Pendiente.

## 5. Distancias de cableado horizontal y cálculo de bobinas

Pendiente.

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
