---
title: "Guía de Repositorios de Charts"
description: "Cómo crear y trabajar con repositorios de charts de Helm."
aliases: ["/docs/chart_repository/"]
weight: 6
---

Esta sección explica cómo crear y trabajar con repositorios de charts de Helm. A
alto nivel, un repositorio de charts es una ubicación donde se pueden almacenar
y compartir charts empaquetados.

El repositorio de charts de la comunidad distribuida de Helm se encuentra en
[Artifact Hub](https://artifacthub.io/packages/search?kind=0) y da la bienvenida
a la participación. Pero Helm también hace posible crear y ejecutar tu propio
repositorio de charts. Esta guía explica cómo hacerlo. Si estás considerando
crear un repositorio de charts, es posible que quieras considerar usar un
[registro OCI]({{< ref "/docs/topics/registries.md" >}}) en su lugar.

## Prerrequisitos

* Revisar la [Guía de Inicio Rápido]({{< ref "quickstart.md" >}})
* Leer el documento de [Charts]({{< ref "/docs/topics/charts.md" >}})

## Crear un repositorio de charts

Un _repositorio de charts_ es un servidor HTTP que aloja un archivo `index.yaml` y
opcionalmente algunos charts empaquetados. Cuando estés listo para compartir tus
charts, la forma preferida de hacerlo es subiéndolos a un repositorio de charts.

A partir de Helm 2.2.0, se admite la autenticación SSL del lado del cliente a un
repositorio. Otros protocolos de autenticación pueden estar disponibles como
plugins.

Debido a que un repositorio de charts puede ser cualquier servidor HTTP que pueda
servir archivos YAML y tar y pueda responder a solicitudes GET, tienes una
gran cantidad de opciones cuando se trata de alojar tu propio repositorio de
charts. Por ejemplo, puedes usar un bucket de Google Cloud Storage (GCS), un
bucket de Amazon S3, GitHub Pages, o incluso crear tu propio servidor web.

### La estructura del repositorio de charts

Un repositorio de charts consiste en charts empaquetados y un archivo especial
llamado `index.yaml` que contiene un índice de todos los charts en el
repositorio. Frecuentemente, los charts que `index.yaml` describe también están
alojados en el mismo servidor, al igual que los [archivos de
procedencia]({{< ref "provenance.md" >}}).

Por ejemplo, la estructura del repositorio `https://example.com/charts` podría
verse así:

```
charts/
  |
  |- index.yaml
  |
  |- alpine-0.1.2.tgz
  |
  |- alpine-0.1.2.tgz.prov
```

En este caso, el archivo de índice contendría información sobre un chart, el
chart Alpine, y proporcionaría la URL de descarga
`https://example.com/charts/alpine-0.1.2.tgz` para ese chart.

No es necesario que un paquete de chart esté ubicado en el mismo servidor que el
archivo `index.yaml`. Sin embargo, hacerlo suele ser lo más fácil.

### El archivo de índice

El archivo de índice es un archivo yaml llamado `index.yaml`. Contiene algunos
metadatos sobre el paquete, incluyendo el contenido del archivo `Chart.yaml` de
un chart. Un repositorio de charts válido debe tener un archivo de índice. El
archivo de índice contiene información sobre cada chart en el repositorio de
charts. El comando `helm repo index` generará un archivo de índice basado en un
directorio local dado que contiene charts empaquetados.

Este es un ejemplo de un archivo de índice:

```yaml
apiVersion: v1
entries:
  alpine:
    - created: 2016-10-06T16:23:20.499814565-06:00
      description: Despliega un pod básico de Alpine Linux
      digest: 99c76e403d752c84ead610644d4b1c2f2b453a74b921f422b9dcb8a7c8b559cd
      home: https://helm.sh/helm
      name: alpine
      sources:
      - https://github.com/helm/helm
      urls:
      - https://technosophos.github.io/tscharts/alpine-0.2.0.tgz
      version: 0.2.0
    - created: 2016-10-06T16:23:20.499543808-06:00
      description: Despliega un pod básico de Alpine Linux
      digest: 515c58e5f79d8b2913a10cb400ebb6fa9c77fe813287afbacf1a0b897cd78727
      home: https://helm.sh/helm
      name: alpine
      sources:
      - https://github.com/helm/helm
      urls:
      - https://technosophos.github.io/tscharts/alpine-0.1.0.tgz
      version: 0.1.0
  nginx:
    - created: 2016-10-06T16:23:20.499543808-06:00
      description: Crea un servidor HTTP nginx básico
      digest: aaff4545f79d8b2913a10cb400ebb6fa9c77fe813287afbacf1a0b897cdffffff
      home: https://helm.sh/helm
      name: nginx
      sources:
      - https://github.com/helm/charts
      urls:
      - https://technosophos.github.io/tscharts/nginx-1.1.0.tgz
      version: 1.1.0
generated: 2016-10-06T16:23:20.499029981-06:00
```

## Alojar Repositorios de Charts

Esta parte muestra varias formas de servir un repositorio de charts.

### Google Cloud Storage

El primer paso es **crear tu bucket de GCS**. Llamaremos al nuestro
`fantastic-charts`.

![Crear un Bucket de GCS](https://helm.sh/img/create-a-bucket.png)

A continuación, haz público tu bucket **editando los permisos del bucket**.

![Editar Permisos](https://helm.sh/img/edit-permissions.png)

Inserta esta línea para **hacer público tu bucket**:

![Hacer Público el Bucket](https://helm.sh/img/make-bucket-public.png)

¡Felicitaciones, ahora tienes un bucket de GCS vacío listo para servir charts!

Puedes subir tu repositorio de charts usando la herramienta de línea de comandos
de Google Cloud Storage, o usando la interfaz web de GCS. Un bucket público de
GCS puede ser accedido vía HTTPS simple en esta dirección:
`https://bucket-name.storage.googleapis.com/`.

### Cloudsmith

También puedes configurar repositorios de charts usando Cloudsmith. Lee más sobre
repositorios de charts con Cloudsmith
[aquí](https://help.cloudsmith.io/docs/helm-chart-repository)

### JFrog Artifactory

De manera similar, también puedes configurar repositorios de charts usando JFrog
Artifactory. Lee más sobre repositorios de charts con JFrog Artifactory
[aquí](https://www.jfrog.com/confluence/display/RTF/Helm+Chart+Repositories)

### Ejemplo de GitHub Pages

De manera similar puedes crear un repositorio de charts usando GitHub Pages.

GitHub te permite servir páginas web estáticas de dos maneras diferentes:

- Configurando un proyecto para servir el contenido de su directorio `docs/`
- Configurando un proyecto para servir una rama particular

Tomaremos el segundo enfoque, aunque el primero es igual de fácil.

El primer paso será **crear tu rama gh-pages**. Puedes hacerlo localmente como:

```console
$ git checkout -b gh-pages
```

O vía navegador web usando el botón **Branch** en tu repositorio de GitHub:

![Crear rama de GitHub Pages](https://helm.sh/img/create-a-gh-page-button.png)

A continuación, querrás asegurarte de que tu **rama gh-pages** esté configurada
como GitHub Pages, haz clic en **Settings** de tu repositorio y desplázate hacia
abajo hasta la sección **GitHub pages** y configura como se muestra a
continuación:

![Crear rama de GitHub Pages](https://helm.sh/img/set-a-gh-page.png)

Por defecto, **Source** generalmente se establece en **rama gh-pages**. Si esto
no está establecido por defecto, entonces selecciónalo.

Puedes usar un **dominio personalizado** allí si lo deseas.

Y verifica que **Enforce HTTPS** esté marcado, para que se use **HTTPS** cuando
se sirvan los charts.

En tal configuración puedes usar tu rama predeterminada para almacenar tu código
de charts, y la **rama gh-pages** como repositorio de charts, por ejemplo:
`https://USERNAME.github.io/REPONAME`. El repositorio de demostración [TS
Charts](https://github.com/technosophos/tscharts) es accesible en
`https://technosophos.github.io/tscharts/`.

Si has decidido usar GitHub pages para alojar el repositorio de charts, revisa
[Chart Releaser Action]({{< ref "/docs/howto/chart_releaser_action.md" >}}).
Chart Releaser Action es un flujo de trabajo de GitHub Action para convertir un
proyecto de GitHub en un repositorio de charts de Helm autoalojado, usando la
herramienta CLI [helm/chart-releaser](https://github.com/helm/chart-releaser).

### Servidores web ordinarios

Para configurar un servidor web ordinario para servir charts de Helm, solo
necesitas hacer lo siguiente:

- Coloca tu índice y charts en un directorio que el servidor pueda servir
- Asegúrate de que el archivo `index.yaml` pueda ser accedido sin requisitos de
  autenticación
- Asegúrate de que los archivos `yaml` se sirvan con el tipo de contenido
  correcto (`text/yaml` o `text/x-yaml`)

Por ejemplo, si quieres servir tus charts desde `$WEBROOT/charts`, asegúrate de
que haya un directorio `charts/` en tu raíz web, y coloca el archivo de índice y
los charts dentro de esa carpeta.

### Servidor de Repositorio ChartMuseum

ChartMuseum es un servidor de Repositorio de Charts de Helm de código abierto
escrito en Go (Golang), con soporte para backends de almacenamiento en la nube,
incluyendo [Google Cloud Storage](https://cloud.google.com/storage/), [Amazon
S3](https://aws.amazon.com/s3/), [Microsoft Azure Blob
Storage](https://azure.microsoft.com/en-us/services/storage/blobs/), [Alibaba
Cloud OSS Storage](https://www.alibabacloud.com/product/oss), [Openstack Object
Storage](https://developer.openstack.org/api-ref/object-store/), [Oracle Cloud
Infrastructure Object Storage](https://cloud.oracle.com/storage), [Baidu Cloud
BOS Storage](https://cloud.baidu.com/product/bos.html), [Tencent Cloud Object
Storage](https://intl.cloud.tencent.com/product/cos), [DigitalOcean
Spaces](https://www.digitalocean.com/products/spaces/),
[Minio](https://min.io/), y [etcd](https://etcd.io/).

También puedes usar el servidor
[ChartMuseum](https://chartmuseum.com/docs/#using-with-local-filesystem-storage)
para alojar un repositorio de charts desde un sistema de archivos local.

### Registro de Paquetes de GitLab

Con GitLab puedes publicar charts de Helm en el Registro de Paquetes de tu
proyecto. Lee más sobre cómo configurar un repositorio de paquetes de helm con
GitLab [aquí](https://docs.gitlab.com/ee/user/packages/helm_repository/).

## Gestionar Repositorios de Charts

Ahora que tienes un repositorio de charts, la última parte de esta guía explica
cómo mantener los charts en ese repositorio.

### Almacenar charts en tu repositorio de charts

Ahora que tienes un repositorio de charts, vamos a subir un chart y un archivo de
índice al repositorio. Los charts en un repositorio de charts deben estar
empaquetados (`helm package chart-name/`) y versionados correctamente (siguiendo
las directrices de [SemVer 2](https://semver.org/)). 

Estos próximos pasos componen un flujo de trabajo de ejemplo, pero eres libre de
usar cualquier flujo de trabajo que prefieras para almacenar y actualizar charts
en tu repositorio de charts.

Una vez que tengas un chart empaquetado listo, crea un nuevo directorio y mueve
tu chart empaquetado a ese directorio.

```console
$ helm package docs/examples/alpine/
$ mkdir fantastic-charts
$ mv alpine-0.1.0.tgz fantastic-charts/
$ helm repo index fantastic-charts --url https://fantastic-charts.storage.googleapis.com
```

El último comando toma la ruta del directorio local que acabas de crear y la URL
de tu repositorio de charts remoto y compone un archivo `index.yaml` dentro de
la ruta del directorio dada.

Ahora puedes subir el chart y el archivo de índice a tu repositorio de charts
usando una herramienta de sincronización o manualmente. Si estás usando Google
Cloud Storage, revisa este [ejemplo de flujo de
trabajo]({{< ref "/docs/howto/chart_repository_sync_example.md" >}}) usando el
cliente gsutil. Para GitHub, simplemente puedes colocar los charts en la rama de
destino apropiada.

### Agregar nuevos charts a un repositorio existente

Cada vez que quieras agregar un nuevo chart a tu repositorio, debes regenerar el
índice. El comando `helm repo index` reconstruirá completamente el archivo
`index.yaml` desde cero, incluyendo solo los charts que encuentra localmente.

Sin embargo, puedes usar la bandera `--merge` para agregar incrementalmente
nuevos charts a un archivo `index.yaml` existente (una gran opción cuando
trabajas con un repositorio remoto como GCS). Ejecuta `helm repo index --help`
para aprender más.

Asegúrate de subir tanto el archivo `index.yaml` revisado como el chart. Y si
generaste un archivo de procedencia, súbelo también.

### Compartir tus charts con otros

Cuando estés listo para compartir tus charts, simplemente hazle saber a alguien
cuál es la URL de tu repositorio.

A partir de ahí, agregarán el repositorio a su cliente helm mediante el comando
`helm repo add [NOMBRE] [URL]` con cualquier nombre que deseen usar para
referenciar el repositorio.

```console
$ helm repo add fantastic-charts https://fantastic-charts.storage.googleapis.com
$ helm repo list
fantastic-charts    https://fantastic-charts.storage.googleapis.com
```

Si los charts están respaldados por autenticación HTTP básica, también puedes
proporcionar el nombre de usuario y la contraseña aquí:

```console
$ helm repo add fantastic-charts https://fantastic-charts.storage.googleapis.com --username my-username --password my-password
$ helm repo list
fantastic-charts    https://fantastic-charts.storage.googleapis.com
```

**Nota:** Un repositorio no se agregará si no contiene un `index.yaml` válido.

**Nota:** Si tu repositorio de helm está usando, por ejemplo, un certificado
autofirmado, puedes usar `helm repo add --insecure-skip-tls-verify ...` para
omitir la verificación de CA.

Después de eso, tus usuarios podrán buscar entre tus charts. Después de que
hayas actualizado el repositorio, pueden usar el comando `helm repo update`
para obtener la información más reciente de los charts.

*Internamente, los comandos `helm repo add` y `helm repo update` están
obteniendo el archivo index.yaml y almacenándolos en el directorio
`$XDG_CACHE_HOME/helm/repository/cache/`. Aquí es donde la función `helm
search` encuentra información sobre los charts.* 
