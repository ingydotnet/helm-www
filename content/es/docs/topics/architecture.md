---
title: "Arquitectura de Helm"
description: "Describe la arquitectura de Helm a alto nivel."
aliases: ["/docs/architecture/"]
weight: 8
---

# Arquitectura de Helm

Este documento describe la arquitectura de Helm a alto nivel.

## El Propósito de Helm

Helm es una herramienta para gestionar paquetes de Kubernetes llamados _charts_.
Helm puede hacer lo siguiente:

- Crear nuevos charts desde cero
- Empaquetar charts en archivos de archivo de chart (tgz)
- Interactuar con repositorios de charts donde se almacenan los charts
- Instalar y desinstalar charts en un clúster de Kubernetes existente
- Gestionar el ciclo de vida de los charts que han sido instalados con Helm

Para Helm, hay tres conceptos importantes:

1. El _chart_ es un conjunto de información necesaria para crear una instancia de
   una aplicación Kubernetes.
2. La _configuración_ contiene información de configuración que puede fusionarse
   con un chart empaquetado para crear un objeto liberable.
3. Un _lanzamiento_ es una instancia en ejecución de un _chart_, combinada con
   una _configuración_ específica.

## Componentes

Helm es un ejecutable que se implementa en dos partes distintas:

**El Cliente Helm** es un cliente de línea de comandos para usuarios finales.
El cliente es responsable de lo siguiente:

- Desarrollo local de charts
- Gestión de repositorios
- Gestión de lanzamientos
- Interfaz con la biblioteca de Helm
  - Envío de charts para ser instalados
  - Solicitud de actualización o desinstalación de lanzamientos existentes

**La Biblioteca Helm** proporciona la lógica para ejecutar todas las operaciones
de Helm. Se comunica con el servidor API de Kubernetes y proporciona la siguiente
capacidad:

- Combinar un chart y configuración para construir un lanzamiento
- Instalar charts en Kubernetes y proporcionar el objeto de lanzamiento
  subsiguiente
- Actualizar y desinstalar charts interactuando con Kubernetes

La biblioteca independiente de Helm encapsula la lógica de Helm para que pueda
ser aprovechada por diferentes clientes.

## Implementación

El cliente y la biblioteca de Helm están escritos en el lenguaje de programación
Go.

La biblioteca utiliza la biblioteca cliente de Kubernetes para comunicarse con
Kubernetes. Actualmente, esa biblioteca utiliza REST+JSON. Almacena información
en Secrets ubicados dentro de Kubernetes. No necesita su propia base de datos.

Los archivos de configuración están, cuando es posible, escritos en YAML.
