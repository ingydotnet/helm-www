---
title: "Usar registros basados en OCI"
description: "Describe cómo usar OCI para la distribución de Charts."
aliases: ["/docs/registries/"]
weight: 7
---

A partir de Helm 3, puede usar registros de contenedores con soporte [OCI](https://www.opencontainers.org/) para almacenar y compartir paquetes de charts. A partir de Helm v3.8.0, el soporte OCI está habilitado por defecto.

## Soporte OCI antes de v3.8.0

El soporte OCI pasó de experimental a disponibilidad general con Helm v3.8.0. En versiones anteriores de Helm, el soporte OCI se comportaba de manera diferente. Si estaba usando soporte OCI antes de Helm v3.8.0, es importante entender qué ha cambiado con las diferentes versiones de Helm.

### Habilitar soporte OCI antes de v3.8.0

Antes de Helm v3.8.0, el soporte OCI es *experimental* y debe ser habilitado.

Para habilitar el soporte experimental OCI para versiones de Helm anteriores a v3.8.0, establezca `HELM_EXPERIMENTAL_OCI` en su entorno. Por ejemplo:

```console
export HELM_EXPERIMENTAL_OCI=1
```

### Deprecación de características OCI y cambios de comportamiento con v3.8.0

Con el lanzamiento de [Helm v3.8.0](https://github.com/helm/helm/releases/tag/v3.8.0), las siguientes características y comportamientos son diferentes de las versiones anteriores de Helm:

- Al establecer un chart en las dependencias como OCI, la versión puede establecerse como un rango como otras dependencias.
- Se pueden subir y usar etiquetas SemVer que incluyen información de compilación. Los registros OCI no soportan `+` como carácter de etiqueta. Helm traduce el `+` a `_` cuando se almacena como etiqueta.
- El comando `helm registry login` ahora sigue la misma estructura que la CLI de Docker para almacenar credenciales. La misma ubicación para la configuración del registro puede pasarse tanto a Helm como a la CLI de Docker.

### Deprecación de características OCI y cambios de comportamiento con v3.7.0

El lanzamiento de [Helm v3.7.0](https://github.com/helm/helm/releases/tag/v3.7.0) incluyó la implementación de [HIP 6](https://github.com/helm/community/blob/main/hips/hip-0006.md) para soporte OCI. Como resultado, las siguientes características y comportamientos son diferentes de las versiones anteriores de Helm:

- El subcomando `helm chart` ha sido eliminado.
- La caché de charts ha sido eliminada (no hay `helm chart list` etc.).
- Las referencias a registros OCI ahora siempre están prefijadas con `oci://`.
- El nombre base de la referencia al registro debe *siempre* coincidir con el nombre del chart.
- La etiqueta de la referencia al registro debe *siempre* coincidir con la versión semántica del chart (es decir, no hay etiquetas `latest`).
- El tipo de medio de la capa del chart cambió de `application/tar+gzip` a `application/vnd.cncf.helm.chart.content.v1.tar+gzip`.

## Usar un registro basado en OCI

### Repositorios Helm en registros basados en OCI

Un [repositorio Helm]({{< ref "chart_repository.md" >}}) es una forma de alojar y distribuir charts de Helm empaquetados. Un registro basado en OCI puede contener cero o más repositorios Helm y cada uno de esos repositorios puede contener cero o más charts de Helm empaquetados.

### Usar registros alojados

Hay varios registros de contenedores alojados con soporte OCI que puede usar para sus charts de Helm. Por ejemplo:

- [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/push-oci-artifact.html)
- [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-helm-repos#push-chart-to-registry-as-oci-artifact)
- [Docker Hub](https://docs.docker.com/docker-hub/oci-artifacts/)
- [Google Artifact Registry](https://cloud.google.com/artifact-registry/docs/helm/manage-charts)
- [Harbor](https://goharbor.io/docs/main/administration/user-defined-oci-artifact/)
- [IBM Cloud Container Registry](https://cloud.ibm.com/docs/Registry?topic=Registry-registry_helm_charts)
- [JFrog Artifactory](https://jfrog.com/help/r/jfrog-artifactory-documentation/helm-oci-repositories)

Siga la documentación del proveedor del registro de contenedores alojado para crear y configurar un registro con soporte OCI.

**Nota:** Puede ejecutar [Docker Registry](https://docs.docker.com/registry/deploying/) o [`zot`](https://github.com/project-zot/zot), que son registros basados en OCI, en su computadora de desarrollo. Ejecutar un registro basado en OCI en su computadora de desarrollo solo debe usarse para fines de prueba.

### Usar sigstore para firmar charts basados en OCI

El plugin [`helm-sigstore`](https://github.com/sigstore/helm-sigstore) permite usar [Sigstore](https://sigstore.dev/) para firmar charts de Helm con las mismas herramientas utilizadas para firmar imágenes de contenedores. Esto proporciona una alternativa a la [procedencia basada en GPG]({{< ref "provenance.md" >}}) soportada por los [repositorios de charts]({{< ref "chart_repository.md" >}}) clásicos.

Para más detalles sobre el uso del plugin `helm sigstore`, consulte [la documentación de ese proyecto](https://github.com/sigstore/helm-sigstore/blob/main/USAGE.md).

## Comandos para trabajar con registros

### El subcomando `registry`

#### `login`

iniciar sesión en un registro (con entrada manual de contraseña)

```console
$ helm registry login -u myuser localhost:5000
Password:
Login succeeded
```

#### `logout`

cerrar sesión de un registro

```console
$ helm registry logout localhost:5000
Logout succeeded
```

### El subcomando `push`

Subir un chart a un registro basado en OCI:

```console
$ helm push mychart-0.1.0.tgz oci://localhost:5000/helm-charts
Pushed: localhost:5000/helm-charts/mychart:0.1.0
Digest: sha256:ec5f08ee7be8b557cd1fc5ae1a0ac985e8538da7c93f51a51eff4b277509a723
```

El subcomando `push` solo puede usarse contra archivos `.tgz` creados previamente usando `helm package`.

Cuando se usa `helm push` para subir un chart a un registro OCI, la referencia debe estar prefijada con `oci://` y no debe contener el nombre base o la etiqueta.

El nombre base de la referencia al registro se infiere del nombre del chart, y la etiqueta se infiere de la versión semántica del chart. Este es actualmente un requisito estricto.

Ciertos registros requieren que el repositorio y/o el espacio de nombres (si se especifica) se creen de antemano. De lo contrario, se producirá un error durante la operación `helm push`.

Si ha creado un [archivo de procedencia]({{< ref "provenance.md" >}}) (`.prov`), y está presente junto al archivo `.tgz` del chart, se subirá automáticamente al registro al hacer `push`. Esto resulta en una capa extra en [el manifiesto del chart de Helm](#helm-chart-manifest).

Los usuarios del [plugin helm-push](https://github.com/chartmuseum/helm-push) (para subir charts a [ChartMuseum]({{< ref "chart_repository.md" >}}#chartmuseum-repository-server)) pueden experimentar problemas, ya que el plugin entra en conflicto con el nuevo `push` incorporado. A partir de la versión v0.10.0, el plugin ha sido renombrado a `cm-push`.

### Otros subcomandos

El soporte para el protocolo `oci://` también está disponible en varios otros subcomandos. Aquí está la lista completa:

- `helm pull`
- `helm show`
- `helm template`
- `helm install`
- `helm upgrade`

El nombre base (nombre del chart) de la referencia al registro *sí* se incluye para cualquier tipo de acción que implique la descarga del chart (vs. `helm push` donde se omite).

Aquí hay algunos ejemplos de uso de los subcomandos listados anteriormente contra charts basados en OCI:

```
$ helm pull oci://localhost:5000/helm-charts/mychart --version 0.1.0
Pulled: localhost:5000/helm-charts/mychart:0.1.0
Digest: sha256:0be7ec9fb7b962b46d81e4bb74fdcdb7089d965d3baca9f85d64948b05b402ff

$ helm show all oci://localhost:5000/helm-charts/mychart --version 0.1.0
apiVersion: v2
appVersion: 1.16.0
description: A Helm chart for Kubernetes
name: mychart
...

$ helm template myrelease oci://localhost:5000/helm-charts/mychart --version 0.1.0
---
# Source: mychart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
...

$ helm install myrelease oci://localhost:5000/helm-charts/mychart --version 0.1.0
NAME: myrelease
LAST DEPLOYED: Wed Oct 27 15:11:40 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
...

$ helm upgrade myrelease oci://localhost:5000/helm-charts/mychart --version 0.2.0
Release "myrelease" has been upgraded. Happy Helming!
NAME: myrelease
LAST DEPLOYED: Wed Oct 27 15:12:05 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
...
```

## Especificar dependencias

Las dependencias de un chart pueden ser descargadas de un registro usando el subcomando `dependency update`.

El `repository` para una entrada dada en `Chart.yaml` se especifica como la referencia al registro sin el nombre base:

```
dependencies:
  - name: mychart
    version: "2.7.0"
    repository: "oci://localhost:5000/myrepo"
```
Esto descargará `oci://localhost:5000/myrepo/mychart:2.7.0` cuando se ejecute `dependency update`.

## Manifiesto del chart de Helm

Ejemplo de manifiesto de chart de Helm como se representa en un registro
(note los campos `mediaType`):
```json
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.cncf.helm.config.v1+json",
    "digest": "sha256:8ec7c0f2f6860037c19b54c3cfbab48d9b4b21b485a93d87b64690fdb68c2111",
    "size": 117
  },
  "layers": [
    {
      "mediaType": "application/vnd.cncf.helm.chart.content.v1.tar+gzip",
      "digest": "sha256:1b251d38cfe948dfc0a5745b7af5ca574ecb61e52aed10b19039db39af6e1617",
      "size": 2487
    }
  ]
}
```

El siguiente ejemplo contiene un [archivo de procedencia]({{< ref "provenance.md" >}}) (note la capa extra):

```json
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.cncf.helm.config.v1+json",
    "digest": "sha256:8ec7c0f2f6860037c19b54c3cfbab48d9b4b21b485a93d87b64690fdb68c2111",
    "size": 117
  },
  "layers": [
    {
      "mediaType": "application/vnd.cncf.helm.chart.content.v1.tar+gzip",
      "digest": "sha256:1b251d38cfe948dfc0a5745b7af5ca574ecb61e52aed10b19039db39af6e1617",
      "size": 2487
    },
    {
      "mediaType": "application/vnd.cncf.helm.chart.provenance.v1.prov",
      "digest": "sha256:3e207b409db364b595ba862cdc12be96dcdad8e36c59a03b7b3b61c946a5741a",
      "size": 643
    }
  ]
}
```

## Migrar desde repositorios de charts 
