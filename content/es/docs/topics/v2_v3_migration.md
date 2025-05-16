---
title: "Migración de Helm v2 a v3"
description: "Aprenda cómo migrar de Helm v2 a v3."
weight: 13
---

Esta guía muestra cómo migrar de Helm v2 a v3. Helm v2 debe estar instalado y gestionando releases en uno o más clústeres.

## Resumen de los cambios en Helm 3

La lista completa de cambios de Helm 2 a 3 está documentada en la [sección de preguntas frecuentes](https://v3.helm.sh/docs/faq/#changes-since-helm-2). A continuación se muestra un resumen de algunos de esos cambios que un usuario debe conocer antes y durante la migración:

1. Eliminación de Tiller:
   - Reemplaza la arquitectura cliente/servidor por cliente/librería (solo el binario `helm`)
   - La seguridad ahora es por usuario (delegada a la seguridad de usuarios del clúster de Kubernetes)
   - Los releases ahora se almacenan como secrets en el clúster y el metadato del objeto release ha cambiado
   - Los releases se persisten por espacio de nombres de release y ya no en el espacio de nombres de Tiller
2. Repositorio de charts actualizado:
   - `helm search` ahora soporta búsquedas tanto en repositorios locales como consultas en Artifact Hub
3. apiVersion de charts incrementado a "v2" por los siguientes cambios de especificación:
   - Las dependencias de charts enlazadas dinámicamente se movieron a `Chart.yaml` (`requirements.yaml` eliminado y requirements --> dependencies)
   - Los charts de librería (charts helper/comunes) ahora pueden añadirse como dependencias enlazadas dinámicamente
   - Los charts tienen un campo de metadatos `type` para definir si el chart es de tipo `application` o `library`. Por defecto es application, lo que significa que es renderizable e instalable
   - Los charts de Helm 2 (apiVersion=v1) aún se pueden instalar
4. Se añadió la especificación de directorios XDG:
   - Se eliminó Helm home y se reemplazó por la especificación de directorios XDG para almacenar archivos de configuración
   - Ya no es necesario inicializar Helm
   - `helm init` y `helm home` eliminados
5. Cambios adicionales:
   - La instalación/configuración de Helm se simplifica:
     - Solo cliente Helm (binario helm) (sin Tiller)
     - Paradigma run-as-is
   - Los repositorios `local` o `stable` no se configuran por defecto
   - El hook `crd-install` se eliminó y se reemplazó por el directorio `crds` en el chart, donde todos los CRDs definidos se instalarán antes de cualquier renderizado del chart
   - El valor de la anotación del hook `test-failure` se eliminó y `test-success` está en desuso. Use `test` en su lugar
   - Comandos eliminados/reemplazados/agregados:
       - delete --> uninstall : elimina todo el historial del release por defecto (antes se necesitaba `--purge`)
       - fetch --> pull
       - home (eliminado)
       - init (eliminado)
       - install: requiere nombre de release o el argumento `--generate-name`
       - inspect --> show
       - reset (eliminado)
       - serve (eliminado)
       - template: el argumento `-x`/`--execute` se renombró a `-s`/`--show-only`
       - upgrade: se añadió el argumento `--history-max` que limita el número máximo de revisiones guardadas por release (0 para sin límite)
   - La librería Go de Helm 3 ha cambiado mucho y no es compatible con la de Helm 2
   - Los binarios de release ahora se alojan en `get.helm.sh`

## Casos de uso de migración

Los casos de uso de migración son los siguientes:

1. Helm v2 y v3 gestionando el mismo clúster:
   - Este caso solo se recomienda si planea eliminar Helm v2 gradualmente y no requiere que v3 gestione releases desplegados por v2. Todos los nuevos releases deben ser desplegados por v3 y los releases existentes desplegados por v2 deben ser actualizados/eliminados solo por v2
   - Helm v2 y v3 pueden gestionar el mismo clúster sin problemas. Las versiones de Helm pueden instalarse en el mismo o en diferentes sistemas
   - Si instala Helm v3 en el mismo sistema, debe realizar un paso adicional para asegurar que ambas versiones del cliente puedan coexistir hasta que esté listo para eliminar el cliente de Helm v2. Renombre o coloque el binario de Helm v3 en una carpeta diferente para evitar conflictos
   - No hay otros conflictos entre ambas versiones debido a las siguientes diferencias:
     - El almacenamiento de releases (historial) de v2 y v3 es independiente. Los cambios incluyen el recurso de Kubernetes usado para el almacenamiento y los metadatos del objeto release contenidos en el recurso. Los releases también serán por espacio de nombres de usuario en vez de usar el espacio de nombres de Tiller (por ejemplo, el espacio de nombres Tiller por defecto de v2 es kube-system). v2 usa "ConfigMaps" o "Secrets" bajo el espacio de nombres de Tiller y propiedad de `TILLER`. v3 usa "Secrets" en el espacio de nombres de usuario y propiedad de `helm`. Los releases son incrementales en ambas versiones
     - El único problema podría ser si recursos de ámbito de clúster de Kubernetes (por ejemplo, `clusterroles.rbac`) se definen en un chart. El despliegue con v3 fallará aunque sea único en el espacio de nombres, ya que los recursos entrarían en conflicto
     - La configuración de v3 ya no usa `$HELM_HOME` y usa la especificación de directorios XDG. También se crea sobre la marcha según sea necesario. Por tanto, es independiente de la configuración de v2. Esto solo aplica cuando ambas versiones están instaladas en el mismo sistema

2. Migrar de Helm v2 a Helm v3:
   - Este caso aplica cuando desea que Helm v3 gestione releases existentes de Helm v2
   - Debe tener en cuenta que un cliente de Helm v2:
     - puede gestionar uno o varios clústeres de Kubernetes
     - puede conectarse a una o varias instancias de Tiller para un clúster
   - Esto significa que debe tener esto en cuenta al migrar, ya que los releases se despliegan en los clústeres por Tiller y su espacio de nombres. Por tanto, debe migrar para cada clúster y cada instancia de Tiller gestionada por la instancia del cliente de Helm v2
   - La ruta de migración de datos recomendada es la siguiente:
     1. Hacer copia de seguridad de los datos de v2
     2. Migrar la configuración de Helm v2
     3. Migrar los releases de Helm v2
     4. Cuando esté seguro de que Helm v3 gestiona todos los datos de Helm v2 (para todos los clústeres e instancias de Tiller de la instancia del cliente de Helm v2) como se espera, entonces limpie los datos de Helm v2
   - El proceso de migración está automatizado por el plugin [2to3](https://github.com/helm/helm-2to3) de Helm v3

## Referencias

   - Plugin [2to3](https://github.com/helm/helm-2to3) de Helm v3
   - Entrada de blog [aquí](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/) explicando el uso del plugin `2to3` con ejemplos 
