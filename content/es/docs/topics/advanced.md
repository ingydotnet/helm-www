---
title: "Técnicas Avanzadas de Helm"
description: "Explica varias características avanzadas para usuarios expertos de Helm"
aliases: ["/docs/advanced_helm_techniques"]
weight: 9
---

Esta sección explica varias características y técnicas avanzadas para usar Helm.
La información en esta sección está destinada a usuarios "expertos" de Helm que
desean realizar personalizaciones y manipulaciones avanzadas de sus charts y
lanzamientos. Cada una de estas características avanzadas viene con sus propios
compromisos y advertencias, por lo que cada una debe usarse con cuidado y con un
conocimiento profundo de Helm. O en otras palabras, recuerda el [principio de
Peter Parker](https://en.wikipedia.org/wiki/With_great_power_comes_great_responsibility)

## Post-Renderizado
El post-renderizado da a los instaladores de charts la capacidad de manipular,
configurar y/o validar manualmente los manifiestos renderizados antes de que
sean instalados por Helm. Esto permite a los usuarios con necesidades de
configuración avanzada poder usar herramientas como
[`kustomize`](https://kustomize.io) para aplicar cambios de configuración sin
necesidad de hacer fork de un chart público o requerir que los mantenedores de
charts especifiquen cada última opción de configuración para una pieza de
software. También hay casos de uso para inyectar herramientas comunes y sidecars
en entornos empresariales o análisis de los manifiestos antes del despliegue.

### Prerrequisitos
- Helm 3.1+

### Uso
Un post-renderizador puede ser cualquier ejecutable que acepte manifiestos de
Kubernetes renderizados en STDIN y devuelva manifiestos de Kubernetes válidos
en STDOUT. Debe devolver un código de salida no-0 en caso de fallo. Esta es la
única "API" entre los dos componentes. Permite una gran flexibilidad en lo que
puedes hacer con tu proceso de post-renderizado.

Un post-renderizador puede usarse con `install`, `upgrade`, y `template`. Para
usar un post-renderizador, usa la bandera `--post-renderer` con una ruta al
ejecutable del renderizador que deseas usar:

```shell
$ helm install mychart stable/wordpress --post-renderer ./path/to/executable
```

Si la ruta no contiene separadores, buscará en $PATH, de lo contrario resolverá
cualquier ruta relativa a una ruta completamente calificada.

Si deseas usar múltiples post-renderizadores, llámalos todos en un script o
juntos en cualquier herramienta binaria que hayas construido. En bash, esto sería
tan simple como `renderer1 | renderer2 | renderer3`.

Puedes ver un ejemplo de uso de `kustomize` como post-renderizador
[aquí](https://github.com/thomastaylor312/advanced-helm-demos/tree/master/post-render).

### Advertencias
Al usar post-renderizadores, hay varias cosas importantes a tener en cuenta. La
más importante es que cuando se usa un post-renderizador, todas las personas que
modifican ese lanzamiento **DEBEN** usar el mismo renderizador para tener
construcciones repetibles. Esta característica está construida intencionalmente
para permitir que cualquier usuario cambie el renderizador que está usando o
deje de usar un renderizador, pero esto debe hacerse deliberadamente para evitar
modificaciones accidentales o pérdida de datos.

Otra nota importante es sobre seguridad. Si estás usando un post-renderizador,
debes asegurarte de que proviene de una fuente confiable (como es el caso de
cualquier otro ejecutable arbitrario). No se recomienda usar renderizadores no
confiables o no verificados ya que tienen acceso completo a las plantillas
renderizadas, que a menudo contienen datos secretos.

### Post-Renderizadores Personalizados
El paso de post-renderizado ofrece aún más flexibilidad cuando se usa en el SDK
de Go. Cualquier post-renderizador solo necesita implementar la siguiente
interfaz de Go:

```go
type PostRenderer interface {
    // Run espera un único buffer lleno con manifiestos renderizados por Helm. Se
    // espera que los resultados modificados sean devueltos en un buffer separado o un
    // error si hubo un problema o fallo mientras se ejecutaba el paso de post-renderizado
    Run(renderedManifests *bytes.Buffer) (modifiedManifests *bytes.Buffer, err error)
}
```

Para más información sobre el uso del SDK de Go, consulta la [sección de SDK de
Go](#go-sdk)

## SDK de Go
Helm 3 debutó con un SDK de Go completamente reestructurado para una mejor
experiencia al construir software y herramientas que aprovechan Helm. La
documentación completa se puede encontrar en la [Sección de SDK de
Go](../sdk/gosdk.md).

## Backends de almacenamiento

Helm 3 cambió el almacenamiento predeterminado de información de lanzamiento a
Secrets en el namespace del lanzamiento. Helm 2 por defecto almacena la
información de lanzamiento como ConfigMaps en el namespace de la instancia de
Tiller. Las subsecciones que siguen muestran cómo configurar diferentes
backends. Esta configuración se basa en la variable de entorno `HELM_DRIVER`.
Puede establecerse a uno de los valores: `[configmap, secret, sql]`.

### Backend de almacenamiento ConfigMap

Para habilitar el backend ConfigMap, necesitarás establecer la variable de
entorno `HELM_DRIVER` a `configmap`.

Puedes establecerla en un shell de la siguiente manera:

```shell
export HELM_DRIVER=configmap
```

Si quieres cambiar del backend predeterminado al backend ConfigMap, tendrás que
hacer la migración por tu cuenta. Puedes recuperar la información del lanzamiento
con el siguiente comando:

```shell
kubectl get secret --all-namespaces -l "owner=helm"
```

**NOTAS DE PRODUCCIÓN**: La información del lanzamiento incluye el contenido de
los archivos de charts y values, y por lo tanto podría contener datos sensibles
(como contraseñas, claves privadas y otras credenciales) que necesitan ser
protegidos de acceso no autorizado. Al gestionar la autorización de Kubernetes,
por ejemplo con [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/),
es posible otorgar acceso más amplio a recursos ConfigMap, mientras se restringe
el acceso a recursos Secret. Por ejemplo, el [rol predeterminado para
usuarios](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)
"view" otorga acceso a la mayoría de los recursos, pero no a Secrets. Además, los
datos de secrets pueden configurarse para [almacenamiento
encriptado](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).
Por favor ten esto en cuenta si decides cambiar al backend ConfigMap, ya que
podría exponer los datos sensibles de tu aplicación.

### Backend de almacenamiento SQL

Hay un backend de almacenamiento SQL ***beta*** que almacena la información de
lanzamiento en una base de datos SQL.

Usar un backend de almacenamiento de este tipo es particularmente útil si tu
información de lanzamiento pesa más de 1MB (en cuyo caso, no puede almacenarse
en ConfigMaps/Secrets debido a límites internos en el almacén de clave-valor
etcd subyacente de Kubernetes).

Para habilitar el backend SQL, necesitarás desplegar una base de datos SQL y
establecer la variable de entorno `HELM_DRIVER` a `sql`. Los detalles de la DB
se establecen con la variable de entorno
`HELM_DRIVER_SQL_CONNECTION_STRING`.

Puedes establecerla en un shell de la siguiente manera:

```shell
export HELM_DRIVER=sql
export HELM_DRIVER_SQL_CONNECTION_STRING=postgresql://helm-postgres:5432/helm?user=helm&password=changeme
```

> Nota: Solo PostgreSQL está soportado en este momento.

**NOTAS DE PRODUCCIÓN**: Se recomienda:
- Hacer que tu base de datos esté lista para producción. Para PostgreSQL, consulta
  la documentación de [Administración del Servidor](https://www.postgresql.org/docs/12/admin.html)
  para más detalles
- Habilitar [gestión de permisos](/docs/permissions_sql_storage_backend/) para
  reflejar RBAC de Kubernetes para la información de lanzamiento

Si quieres cambiar del backend predeterminado al backend SQL, tendrás que hacer
la migración por tu cuenta. Puedes recuperar la información del lanzamiento con
el siguiente comando:

```shell
kubectl get secret --all-namespaces -l "owner=helm"
```
