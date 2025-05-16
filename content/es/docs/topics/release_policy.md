---
title: "Política de programación de lanzamientos"
description: "Describe la política de programación de lanzamientos de Helm."
---

Para beneficio de sus usuarios, Helm define y anuncia las fechas de lanzamiento
con anticipación. Este documento describe la política que rige la programación
de lanzamientos de Helm.

## Calendario de lanzamientos

Un calendario público que muestra los próximos lanzamientos de Helm se puede
encontrar [aquí](https://helm.sh/calendar/release).

## Versionado semántico

Las versiones de Helm se expresan como `x.y.z`, donde `x` es la versión mayor,
`y` es la versión menor, y `z` es la versión de parche, siguiendo la
terminología de [Versionado Semántico](https://semver.org/spec/v2.0.0.html).

## Lanzamientos de parches

Los lanzamientos de parches proporcionan a los usuarios correcciones de errores
y correcciones de seguridad. No contienen nuevas características.

Un nuevo lanzamiento de parche relacionado con el último lanzamiento menor/mayor
normalmente se realizará una vez al mes el segundo miércoles de cada mes.

Un lanzamiento de parche para corregir una regresión de alta prioridad o un
problema de seguridad puede realizarse cuando sea necesario.

Un lanzamiento de parche será cancelado por cualquiera de las siguientes
razones:
- si no hay nuevo contenido desde el lanzamiento anterior
- si la fecha del lanzamiento de parche cae dentro de una semana antes del
  primer candidato a lanzamiento (RC1) de un próximo lanzamiento menor
- si la fecha del lanzamiento de parche cae dentro de las cuatro semanas
  siguientes a un lanzamiento menor

## Lanzamientos menores

Los lanzamientos menores contienen correcciones de seguridad y errores, así
como nuevas características. Son compatibles hacia atrás con respecto a la API
y el uso de la CLI.

Para alinearse con los lanzamientos de Kubernetes, un lanzamiento menor de
Helm se realizará cada 4 meses (3 lanzamientos al año).

Se pueden realizar lanzamientos menores adicionales si es necesario, pero no
afectarán la línea de tiempo de un lanzamiento futuro anunciado, a menos que
el lanzamiento anunciado esté a menos de 7 días.

Al mismo tiempo que se publica un lanzamiento, se anunciará y publicará en la
página web principal de Helm la fecha del próximo lanzamiento menor.

## Lanzamientos mayores

Los lanzamientos mayores contienen cambios que rompen la compatibilidad. Tales
lanzamientos son raros pero a veces son necesarios para permitir que Helm
continúe evolucionando en direcciones nuevas importantes.

Los lanzamientos mayores pueden ser difíciles de planificar. Con eso en mente,
una fecha final de lanzamiento solo se elegirá y anunciará una vez que la
primera versión beta de dicho lanzamiento esté disponible. 
