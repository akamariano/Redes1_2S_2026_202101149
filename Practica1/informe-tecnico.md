# Informe de Desarrollo, Práctica 1, Redes de Computadoras 1

Estudiante: Mariano Roberto Rac Noguera
Carné: 202101149
Curso: Redes de Computadoras 1, Segundo Semestre 2026

URL de cálculos (hoja de trabajo con las tablas de centroide, distancias Manhattan y dimensionamiento): https://ingenieriausacedu-my.sharepoint.com/:x:/g/personal/3530110610101_ingenieria_usac_edu_gt/IQAJAMek7jhgT6Hr3Tu61K0aAfKd5wrboL2uypH5l7NK5yY?e=tSEymB

## Proceso de diseño

La práctica se desarrolló siguiendo un orden lógico. Primero se definió la ubicación del cuarto de telecomunicaciones (MDF), porque de ella dependen las distancias del cableado, la cantidad de bobinas y la selección del medio de transmisión. Después se definieron las topologías, el equipo activo, los estándares de capa física, la documentación, el presupuesto y los diagramas.

## Criterios para la selección de topologías

La ubicación del MDF se determinó mediante un centroide ponderado. Se usó como peso la cantidad de hosts de cada departamento. Este método representa mejor la distribución de los equipos que colocar el MDF en el centro geométrico del edificio. El resultado fue el punto (13.32, 6.88), que cae dentro del Vestíbulo de Ingreso. Se eligió la esquina superior izquierda, junto al Hall Central y fuera del recorrido de la puerta, porque allí existe espacio real para instalar el equipo.

Para elegir la topología se consideraron la cantidad de hosts, la importancia del segmento, el costo, el crecimiento futuro y la tolerancia a fallos. La mayoría de departamentos usa una estrella simple y un enlace troncal sin redundancia, porque no tiene equipo crítico y un solo switch es suficiente. El Data Center es la excepción: concentra los tres servidores principales, por lo que usa dos rutas físicas hacia el switch central. Diseño e Innovación también se divide en dos switches: uno para Diseño UI/UX y Data Analytics, y otro para QA. Esto permite separar el tráfico de pruebas y facilita la solución de fallas con un costo adicional bajo.

## Medios de transmisión y equipo activo

Ninguna distancia del edificio se acerca al límite de 100 metros establecido por TIA/EIA-568 para el cable UTP. Por eso, la selección del cable se basó en el ancho de banda y la importancia de cada segmento. Se eligió Cat6 para los segmentos normales, porque ofrece 1 Gbps a un costo razonable. Para el Data Center se eligió fibra óptica multimodo OM3, tanto para el cableado horizontal como para las dos rutas troncales, porque allí se concentra el tráfico de los servidores principales.

El equipo activo se dimensionó a partir de la cantidad de hosts. Se eligió un patch panel de 48 puertos porque coincide con el total de puntos de red. El switch central tiene 16 puertos: necesita 9 para las conexiones troncales y deja puertos disponibles para crecer. Los switches departamentales se calcularon sumando los hosts y los puertos necesarios para los enlaces, y luego se eligió el siguiente tamaño estándar: 8, 12 o 16 puertos.

## Retos de planificación física al interpretar el plano base

El principal reto fue ubicar correctamente el Data Center en el sistema de coordenadas. Este ambiente está fuera del edificio principal, que mide 26 por 16 metros, y se conecta por un corredor secundario de 1.5 metros. Fue necesario corregir su coordenada x, porque el primer cálculo no tomaba en cuenta el desplazamiento del corredor. De no corregirse, también habrían cambiado el centroide y las distancias del enlace troncal.

Otro reto fue convertir las distancias del plano en metros reales de cable. Los cables siguen la canalización y no una línea recta. Por eso se usó la distancia Manhattan, que suma las diferencias en x y en y. También se agregó un 20% para cubrir las subidas a la canalización del techo y el service loop de cada extremo. Así se obtuvo una estimación más realista del material.

Finalmente, se debía mostrar en el plano el MDF, los puntos de red, los tipos de toma, el cableado horizontal y troncal, la canalización y las etiquetas. Fue más práctico dibujar directamente sobre una copia del plano usando iPad y Apple Pencil. De esta forma se conservaron mejor las proporciones y las distancias del edificio que al recrearlo desde cero en otra herramienta.

## Justificación del medio utilizado en el cableado troncal

En los siete departamentos normales se usó cobre Cat6 para el cableado troncal. Las distancias son cortas y el cable ofrece el ancho de banda necesario para el tráfico de oficina a menor costo que la fibra. En el Data Center se usó fibra multimodo OM3 para las dos rutas troncales. La fibra ofrece más capacidad, resiste mejor la interferencia electromagnética y el costo adicional se limita a un segmento corto y crítico.