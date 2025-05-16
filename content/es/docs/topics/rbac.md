---
title: "Control de Acceso Basado en Roles"
description: "Explica cómo Helm interactúa con el Control de Acceso Basado en Roles de Kubernetes."
aliases: ["/docs/rbac/"]
weight: 11
---

En Kubernetes, otorgar roles a un usuario o a una cuenta de servicio específica
para una aplicación es una mejor práctica para asegurar que su aplicación
esté operando dentro del alcance que ha especificado. Lea más sobre los
permisos de cuentas de servicio [en la documentación oficial de
Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions).

Desde Kubernetes 1.6 en adelante, el Control de Acceso Basado en Roles está
habilitado por defecto. RBAC le permite especificar qué tipos de acciones están
permitidas dependiendo del usuario y su rol en su organización.

Con RBAC, puede:

- otorgar operaciones privilegiadas (crear recursos a nivel de clúster, como
  nuevos roles) a administradores
- limitar la capacidad de un usuario para crear recursos (pods, volúmenes
  persistentes, deployments) a espacios de nombres específicos, o en ámbitos
  de todo el clúster (cuotas de recursos, roles, definiciones de recursos
  personalizados)
- limitar la capacidad de un usuario para ver recursos ya sea en espacios de
  nombres específicos o a nivel de todo el clúster.

Esta guía es para administradores que desean restringir el alcance de la
interacción de un usuario con la API de Kubernetes.

## Gestión de cuentas de usuario

Todos los clústeres de Kubernetes tienen dos categorías de usuarios: cuentas de
servicio gestionadas por Kubernetes y usuarios normales.

Los usuarios normales se asume que son gestionados por un servicio externo e
independiente. Un administrador distribuyendo claves privadas, un almacén de
usuarios como Keystone o Google Accounts, incluso un archivo con una lista de
nombres de usuario y contraseñas. En este sentido, Kubernetes no tiene objetos
que representen cuentas de usuario normales. Los usuarios normales no pueden
ser añadidos a un clúster a través de una llamada API.

En contraste, las cuentas de servicio son usuarios gestionados por la API de
Kubernetes. Están vinculadas a espacios de nombres específicos y son creadas
automáticamente por el servidor API o manualmente a través de llamadas API.
Las cuentas de servicio están vinculadas a un conjunto de credenciales
almacenadas como Secrets, que se montan en los pods permitiendo que los
procesos dentro del clúster se comuniquen con la API de Kubernetes.

Las solicitudes API están vinculadas a un usuario normal o a una cuenta de
servicio, o son tratadas como solicitudes anónimas. Esto significa que cada
proceso dentro o fuera del clúster, desde un usuario humano escribiendo
`kubectl` en una estación de trabajo, hasta los kubelets en los nodos, hasta
los miembros del plano de control, debe autenticarse al hacer solicitudes al
servidor API, o ser tratado como un usuario anónimo.

## Roles, ClusterRoles, RoleBindings y ClusterRoleBindings

En Kubernetes, las cuentas de usuario y las cuentas de servicio solo pueden
ver y editar recursos a los que se les ha otorgado acceso. Este acceso se
otorga a través del uso de Roles y RoleBindings. Los Roles y RoleBindings
están vinculados a un espacio de nombres particular, lo que otorga a los
usuarios la capacidad de ver y/o editar recursos dentro de ese espacio de
nombres al que el Rol les proporciona acceso.

A nivel de clúster, estos se llaman ClusterRoles y ClusterRoleBindings.
Otorgar a un usuario un ClusterRole le da acceso para ver y/o editar recursos
en todo el clúster. También es necesario para ver y/o editar recursos a nivel
de clúster (espacios de nombres, cuotas de recursos, nodos).

Los ClusterRoles pueden estar vinculados a un espacio de nombres particular a
través de una referencia en un RoleBinding. Los ClusterRoles predeterminados
`admin`, `edit` y `view` se usan comúnmente de esta manera.

Estos son algunos ClusterRoles disponibles por defecto en Kubernetes. Están
destinados a ser roles orientados al usuario. Incluyen roles de superusuario
(`cluster-admin`), y roles con acceso más granular (`admin`, `edit`, `view`).

| ClusterRole Predeterminado | ClusterRoleBinding Predeterminado | Descripción
|----------------------------|-----------------------------------|------------
| `cluster-admin`            | grupo `system:masters`            | Permite acceso de superusuario para realizar cualquier acción en cualquier recurso. Cuando se usa en un ClusterRoleBinding, da control total sobre cada recurso en el clúster y en todos los espacios de nombres. Cuando se usa en un RoleBinding, da control total sobre cada recurso en el espacio de nombres del rolebinding, incluyendo el espacio de nombres en sí.
| `admin`                    | Ninguno                          | Permite acceso de administrador, destinado a ser otorgado dentro de un espacio de nombres usando un RoleBinding. Si se usa en un RoleBinding, permite acceso de lectura/escritura a la mayoría de los recursos en un espacio de nombres, incluyendo la capacidad de crear roles y rolebindings dentro del espacio de nombres. No permite acceso de escritura a la cuota de recursos o al espacio de nombres en sí.
| `edit`                     | Ninguno                          | Permite acceso de lectura/escritura a la mayoría de los objetos en un espacio de nombres. No permite ver o modificar roles o rolebindings.
| `view`                     | Ninguno                          | Permite acceso de solo lectura para ver la mayoría de los objetos en un espacio de nombres. No permite ver roles o rolebindings. No permite ver secrets, ya que estos son escalados.

## Restringir el acceso de una cuenta de usuario usando RBAC

Ahora que entendemos los conceptos básicos del Control de Acceso Basado en
Roles, discutamos cómo un administrador puede restringir el alcance de acceso
de un usuario.

### Ejemplo: Otorgar a un usuario acceso de lectura/escritura a un espacio de nombres particular

Para restringir el acceso de un usuario a un espacio de nombres particular,
podemos usar el rol `edit` o el rol `admin`. Si sus charts crean o interactúan
con Roles y Rolebindings, querrá usar el ClusterRole `admin`.

Además, también puede crear un RoleBinding con acceso `cluster-admin`.
Otorgar a un usuario acceso `cluster-admin` a nivel de espacio de nombres
proporciona control total sobre cada recurso en el espacio de nombres,
incluyendo el espacio de nombres en sí.

Para este ejemplo, crearemos un usuario con el Rol `edit`. Primero, cree el
espacio de nombres:

```console
$ kubectl create namespace foo
```

Ahora, cree un RoleBinding en ese espacio de nombres, otorgando al usuario el
rol `edit`.

```console
$ kubectl create rolebinding sam-edit
    --clusterrole edit \
    --user sam \
    --namespace foo
```

### Ejemplo: Otorgar a un usuario acceso de lectura/escritura a nivel de clúster

Si un usuario desea instalar un chart que instala recursos a nivel de clúster
(espacios de nombres, roles, definiciones de recursos personalizados, etc.),
necesitará acceso de escritura a nivel de clúster.

Para hacer eso, otorgue al usuario acceso `admin` o `cluster-admin`.

Otorgar a un usuario acceso `cluster-admin` le da acceso a absolutamente todos
los recursos disponibles en Kubernetes, incluyendo acceso a nodos con
`kubectl drain` y otras tareas administrativas. Se recomienda encarecidamente
considerar proporcionar al usuario acceso `admin` en su lugar, o crear un
ClusterRole personalizado adaptado a sus necesidades.

```console
$ kubectl create clusterrolebinding sam-view
    --clusterrole view \
    --user sam

$ kubectl create clusterrolebinding sam-secret-reader
    --clusterrole secret-reader \
    --user sam
```

### Ejemplo: Otorgar a un usuario acceso de solo lectura a un espacio de nombres particular

Es posible que haya notado que no hay un ClusterRole disponible para ver
secrets. El ClusterRole `view` no otorga a un usuario acceso de lectura a
Secrets debido a preocupaciones de escalado. Helm almacena los metadatos de
release como Secrets por defecto.

Para que un usuario pueda ejecutar `helm list`, necesita poder leer estos
secrets. Para eso, crearemos un ClusterRole especial `secret-reader`.

Cree el archivo `cluster-role-secret-reader.yaml` y escriba el siguiente
contenido en el archivo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

Luego, cree el ClusterRole usando

```console
$ kubectl create -f clusterrole-secret-reader.yaml
```

Una vez hecho esto, podemos otorgar a un usuario acceso de lectura a la
mayoría de los recursos, y luego otorgarle acceso de lectura a secrets:

```console
$ kubectl create namespace foo

$ kubectl create rolebinding sam-view
    --clusterrole view \
    --user sam \
    --namespace foo

$ kubectl create rolebinding sam-secret-reader
    --clusterrole secret-reader \
    --user sam \
    --namespace foo
```

### Ejemplo: Otorgar a un usuario acceso de solo lectura a nivel de clúster

En ciertos escenarios, puede ser beneficioso otorgar a un usuario acceso a
nivel de clúster. Por ejemplo, si un usuario quiere ejecutar el comando
`helm list --all-namespaces`, la API requiere que el usuario tenga acceso de
lectura a nivel de clúster.

Para hacer eso, otorgue al usuario tanto acceso `view` como `secret-reader` como
se describió anteriormente, pero con un ClusterRoleBinding.

```console
$ kubectl create clusterrolebinding sam-view
    --clusterrole view \
    --user sam

$ kubectl create clusterrolebinding sam-secret-reader
    --clusterrole secret-reader \
    --user sam
```

## Consideraciones Adicionales

Los ejemplos mostrados arriba utilizan los ClusterRoles predeterminados
proporcionados con Kubernetes. Para un control más granular sobre qué recursos
se otorgan a los usuarios, consulte [la documentación de
Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
sobre la creación de sus propios Roles y ClusterRoles personalizados. 
