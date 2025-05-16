---
title: "APIs de Kubernetes en desuso"
description: "Explica las APIs de Kubernetes en desuso en Helm"
aliases: ["docs/k8s_apis/"]
---

Kubernetes es un sistema impulsado por API y la API evoluciona con el tiempo para reflejar la comprensión cambiante del espacio de problemas. Esto es una práctica común en los sistemas y sus APIs. Una parte importante de la evolución de las APIs es una buena política y proceso de desaprobación para informar a los usuarios cómo se implementan los cambios en las APIs. En otras palabras, los consumidores de tu API necesitan saber con anticipación y en qué versión una API será eliminada o modificada. Esto elimina el elemento sorpresa y los cambios incompatibles para los consumidores.

La [política de desaprobación de Kubernetes](https://kubernetes.io/docs/reference/using-api/deprecation-policy/) documenta cómo Kubernetes maneja los cambios en sus versiones de API. La política de desaprobación establece el periodo de tiempo durante el cual se soportarán las versiones de API tras un anuncio de desaprobación. Por lo tanto, es importante estar al tanto de los anuncios de desaprobación y saber cuándo se eliminarán las versiones de API, para minimizar el impacto.

Este es un ejemplo de un anuncio [para la eliminación de versiones de API en desuso en Kubernetes 1.16](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/) y se anunció unos meses antes del lanzamiento. Estas versiones de API ya habrían sido anunciadas como en desuso antes de esto. Esto muestra que existe una buena política que informa a los consumidores sobre el soporte de versiones de API.

Las plantillas de Helm especifican un [grupo de API de Kubernetes](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups) al definir un objeto de Kubernetes, similar a un archivo manifiesto de Kubernetes. Se especifica en el campo `apiVersion` de la plantilla e identifica la versión de la API del objeto de Kubernetes. Esto significa que los usuarios de Helm y los mantenedores de charts deben estar atentos cuando las versiones de API de Kubernetes han quedado en desuso y en qué versión de Kubernetes serán eliminadas.

## Mantenedores de charts

Debe auditar sus charts buscando versiones de API de Kubernetes que estén en desuso o hayan sido eliminadas en una versión de Kubernetes. Las versiones de API que estén próximas a quedar fuera de soporte o que ya no estén soportadas deben actualizarse a la versión soportada y publicar una nueva versión del chart. La versión de la API se define por los campos `kind` y `apiVersion`. Por ejemplo, aquí hay una versión eliminada del objeto `Deployment` en Kubernetes 1.16:

```yaml
apiVersion: apps/v1beta1
kind: Deployment
```

## Usuarios de Helm

Debe auditar los charts que utiliza (similar a los [mantenedores de charts](#chart-maintainers)) e identificar cualquier chart donde las versiones de API estén en desuso o eliminadas en una versión de Kubernetes. Para los charts identificados, debe buscar la última versión del chart (que tenga versiones de API soportadas) o actualizar el chart usted mismo.

Además, también debe auditar cualquier chart desplegado (es decir, releases de Helm) verificando nuevamente si hay versiones de API en desuso o eliminadas. Esto se puede hacer obteniendo los detalles de un release usando el comando `helm get manifest`.

La forma de actualizar un release de Helm a APIs soportadas depende de sus hallazgos:

1. Si solo encuentra versiones de API en desuso:
  - Realice un `helm upgrade` con una versión del chart que tenga versiones de API de Kubernetes soportadas
  - Añada una descripción en la actualización, indicando que no se debe hacer rollback a una versión de Helm anterior a la actual
2. Si encuentra alguna(s) versión(es) de API que han sido eliminadas en una versión de Kubernetes:
  - Si está ejecutando una versión de Kubernetes donde las versiones de API aún están disponibles (por ejemplo, está en Kubernetes 1.15 y encontró APIs que serán eliminadas en Kubernetes 1.16):
    - Siga el procedimiento del punto 1
  - De lo contrario (por ejemplo, ya está ejecutando una versión de Kubernetes donde algunas versiones de API reportadas por `helm get manifest` ya no están disponibles):
    - Debe editar el manifiesto del release almacenado en el clúster para actualizar las versiones de API a APIs soportadas. Consulte [Actualizar versiones de API de un manifiesto de release](#actualizar-versiones-de-api-de-un-manifiesto-de-release) para más detalles

> Nota: En todos los casos de actualización de un release de Helm con APIs soportadas, nunca debe hacer rollback del release a una versión anterior a la versión con las APIs soportadas.

> Recomendación: La mejor práctica es actualizar los releases que usan versiones de API en desuso a versiones de API soportadas antes de actualizar a un clúster de Kubernetes que elimine esas versiones de API.

Si no actualiza un release como se sugiere anteriormente, tendrá un error similar al siguiente al intentar actualizar un release en una versión de Kubernetes donde su(s) versión(es) de API han sido eliminadas:

```
Error: UPGRADE FAILED: current release manifest contains removed kubernetes api(s)
for this kubernetes version and it is therefore unable to build the kubernetes
objects for performing the diff. error from kubernetes: unable to recognize "":
no matches for kind "Deployment" in version "apps/v1beta1"
```

Helm falla en este escenario porque intenta crear un diff entre el release desplegado actualmente (que contiene las APIs de Kubernetes que han sido eliminadas en esta versión de Kubernetes) y el chart que está pasando con las versiones de API actualizadas/soportadas. La razón subyacente del fallo es que cuando Kubernetes elimina una versión de API, la librería cliente de Go de Kubernetes ya no puede analizar los objetos en desuso y Helm falla al llamar a la librería. Desafortunadamente, Helm no puede recuperarse de esta situación y ya no puede gestionar ese release. Consulte [Actualizar versiones de API de un manifiesto de release](#actualizar-versiones-de-api-de-un-manifiesto-de-release) para más detalles sobre cómo recuperarse de este escenario.

## Actualizar versiones de API de un manifiesto de release

El manifiesto es una propiedad del objeto release de Helm que se almacena en el campo data de un Secret (por defecto) o ConfigMap en el clúster. El campo data contiene un objeto comprimido con gzip y codificado en base 64 (hay una codificación base 64 adicional para un Secret). Hay un Secret/ConfigMap por versión/revisión de release en el espacio de nombres del release.

Puede usar el plugin de Helm [mapkubeapis](https://github.com/helm/helm-mapkubeapis) para realizar la actualización de un release a APIs soportadas. Consulte el readme para más detalles.

Alternativamente, puede seguir estos pasos manuales para actualizar las versiones de API de un manifiesto de release. Dependiendo de su configuración, siga los pasos para el backend Secret o ConfigMap.

- Obtenga el nombre del Secret o ConfigMap asociado con el release desplegado más reciente:
  - Backend Secrets: `kubectl get secret -l owner=helm,status=deployed,name=<release_name> --namespace <release_namespace> | awk '{print $1}' | grep -v NAME`
  - Backend ConfigMap: `kubectl get configmap -l owner=helm,status=deployed,name=<release_name> --namespace <release_namespace> | awk '{print $1}' | grep -v NAME`
- Obtenga los detalles del release desplegado más reciente:
  - Backend Secrets: `kubectl get secret <release_secret_name> -n <release_namespace> -o yaml > release.yaml`
  - Backend ConfigMap: `kubectl get configmap <release_configmap_name> -n <release_namespace> -o yaml > release.yaml`
- Haga una copia de seguridad del release por si necesita restaurar si algo sale mal:
  - `cp release.yaml release.bak`
  - En caso de emergencia, restaure: `kubectl apply -f release.bak -n <release_namespace>`
- Decodifique el objeto release:
  - Backend Secrets: `cat release.yaml | grep -oP '(?<=release: ).*' | base64 -d | base64 -d | gzip -d > release.data.decoded`
  - Backend ConfigMap: `cat release.yaml | grep -oP '(?<=release: ).*' | base64 -d | gzip -d > release.data.decoded`
- Cambie las versiones de API de los manifiestos. Puede usar cualquier herramienta (por ejemplo, un editor) para hacer los cambios. Esto está en el campo `manifest` de su objeto release decodificado (`release.data.decoded`)
- Codifique el objeto release:
  - Backend Secrets: `cat release.data.decoded | gzip | base64 | base64`
  - Backend ConfigMap: `cat release.data.decoded | gzip | base64`
- Reemplace el valor de la propiedad `data.release` en el archivo del release desplegado (`release.yaml`) con el nuevo objeto release codificado
- Aplique el archivo al espacio de nombres: `kubectl apply -f release.yaml -n <release_namespace>`
- Realice un `helm upgrade` con una versión del chart con versiones de API de Kubernetes soportadas
- Añada una descripción en la actualización, indicando que no se debe hacer rollback a una versión de Helm anterior a la actual 
