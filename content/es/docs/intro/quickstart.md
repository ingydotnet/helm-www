---
title: "Guía de Inicio Rápido"
description: "Cómo instalar y comenzar con Helm, incluyendo instrucciones para distribuciones, preguntas frecuentes y plugins."
weight: 1
aliases: ["/docs/quickstart/"]
---

Esta guía cubre cómo puedes comenzar rápidamente a usar Helm.

## Prerrequisitos

Los siguientes prerrequisitos son necesarios para un uso exitoso y adecuadamente
seguro de Helm.

1. Un clúster de Kubernetes
2. Decidir qué configuraciones de seguridad aplicar a tu instalación, si las hay
3. Instalar y configurar Helm

### Instalar Kubernetes o tener acceso a un clúster

- Debes tener Kubernetes instalado. Para la última versión de Helm, recomendamos
  la última versión estable de Kubernetes, que en la mayoría de los casos es la
  segunda versión menor más reciente.
- También debes tener una copia local configurada de `kubectl`.

Consulta la [Política de Soporte de Versiones de Helm](https://helm.sh/docs/topics/version_skew/) para conocer la máxima diferencia de versión soportada entre Helm y Kubernetes.

## Instalar Helm

Descarga una versión binaria del cliente Helm. Puedes usar herramientas como
`homebrew`, o consultar [la página oficial de lanzamientos](https://github.com/helm/helm/releases).

Para más detalles, o para otras opciones, consulta [la guía de instalación]({{< ref
"install.md" >}}).

## Inicializar un Repositorio de Charts de Helm

Una vez que tengas Helm listo, puedes agregar un repositorio de charts. Consulta
[Artifact Hub](https://artifacthub.io/packages/search?kind=0) para ver los
repositorios de charts de Helm disponibles.

```console
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

Una vez instalado, podrás listar los charts que puedes instalar:

```console
$ helm search repo bitnami
NAME                             	CHART VERSION	APP VERSION  	DESCRIPTION
bitnami/bitnami-common           	0.0.9        	0.0.9        	DEPRECATED Chart with custom templates used in ...
bitnami/airflow                  	8.0.2        	2.0.0        	Apache Airflow is a platform to programmaticall...
bitnami/apache                   	8.2.3        	2.4.46       	Chart for Apache HTTP Server
bitnami/aspnet-core              	1.2.3        	3.1.9        	ASP.NET Core is an open-source framework create...
# ... y muchos más
```

## Instalar un Chart de Ejemplo

Para instalar un chart, puedes ejecutar el comando `helm install`. Helm tiene
varias formas de encontrar e instalar un chart, pero la más fácil es usar los
charts de `bitnami`.

```console
$ helm repo update              # Asegúrate de obtener la última lista de charts
$ helm install bitnami/mysql --generate-name
NAME: mysql-1612624192
LAST DEPLOYED: Sat Feb  6 16:09:56 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES: ...
```

En el ejemplo anterior, el chart `bitnami/mysql` fue lanzado, y el nombre de
nuestro nuevo lanzamiento es `mysql-1612624192`.

Puedes obtener una idea simple de las características de este chart MySQL
ejecutando `helm show chart bitnami/mysql`. O podrías ejecutar `helm show all
bitnami/mysql` para obtener toda la información sobre el chart.

Cada vez que instalas un chart, se crea un nuevo lanzamiento. Por lo tanto, un
chart puede instalarse varias veces en el mismo clúster. Y cada uno puede ser
gestionado y actualizado de forma independiente.

El comando `helm install` es un comando muy potente con muchas capacidades. Para
aprender más sobre él, consulta la [Guía de Uso de Helm]({{< ref "using_helm.md"
>}})

## Aprender Sobre los Lanzamientos

Es fácil ver qué se ha lanzado usando Helm:

```console
$ helm list
NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART      	APP VERSION
mysql-1612624192	default  	1       	2021-02-06 16:09:56.283059 +0100 CET	deployed	mysql-8.3.0	8.0.23
```

La función `helm list` (o `helm ls`) te mostrará una lista de todos los
lanzamientos desplegados.

## Desinstalar un Lanzamiento

Para desinstalar un lanzamiento, usa el comando `helm uninstall`:

```console
$ helm uninstall mysql-1612624192
release "mysql-1612624192" uninstalled
```

Esto desinstalará `mysql-1612624192` de Kubernetes, lo que eliminará todos los
recursos asociados con el lanzamiento, así como el historial del lanzamiento.

Si se proporciona la bandera `--keep-history`, se mantendrá el historial del
lanzamiento. Podrás solicitar información sobre ese lanzamiento:

```console
$ helm status mysql-1612624192
Status: UNINSTALLED
...
```

Debido a que Helm rastrea tus lanzamientos incluso después de haberlos
desinstalado, puedes auditar el historial de un clúster e incluso recuperar un
lanzamiento (con `helm rollback`).

## Leer el Texto de Ayuda

Para aprender más sobre los comandos disponibles de Helm, usa `helm help` o
escribe un comando seguido de la bandera `-h`:

```console
$ helm get -h
```
