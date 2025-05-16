---
title: "Gestión de permisos para el backend de almacenamiento SQL"
description: "Conozca cómo configurar permisos al usar el backend de almacenamiento SQL."
aliases: ["/docs/permissions_sql_storage_backend/"]
---

Este documento tiene como objetivo proporcionar orientación a los usuarios para configurar y gestionar permisos al usar el backend de almacenamiento SQL.

## Introducción

Para gestionar permisos, Helm aprovecha la funcionalidad RBAC de Kubernetes. Cuando se utiliza el backend de almacenamiento SQL, los roles de Kubernetes no pueden usarse para determinar si un usuario puede acceder a un recurso determinado. Este documento muestra cómo crear y gestionar estos permisos.

## Inicialización

La primera vez que la CLI de Helm se conecte a su base de datos, el cliente se asegurará de que haya sido inicializada previamente. Si no es así, se encargará automáticamente de la configuración necesaria. Esta inicialización requiere privilegios de administrador en el esquema público, o al menos la capacidad de:

* crear una tabla
* otorgar privilegios en el esquema público

Después de que la migración se haya ejecutado en su base de datos, todos los demás roles pueden usar el cliente.

## Otorgar privilegios a un usuario no administrador en PostgreSQL

Para gestionar permisos, el driver del backend SQL aprovecha la característica de [RLS](https://www.postgresql.org/docs/9.5/ddl-rowsecurity.html) (Row Level Security) de PostgreSQL. RLS permite que todos los usuarios puedan leer/escribir en la misma tabla, sin poder manipular las mismas filas si no se les permite explícitamente. Por defecto, cualquier rol que no haya recibido explícitamente los privilegios adecuados siempre devolverá una lista vacía al ejecutar `helm list` y no podrá recuperar ni modificar ningún recurso en el clúster.

Veamos cómo otorgar a un rol acceso a espacios de nombres específicos:

```sql
CREATE POLICY <nombre> ON releases_v1 FOR ALL TO <rol> USING (namespace = 'default');
```

Este comando otorgará permisos para leer y escribir todos los recursos que cumplan la condición `namespace = 'default'` al rol `rol`. Después de crear esta política, el usuario que se conecte a la base de datos en nombre del rol `rol` podrá ver todos los releases que existan en el espacio de nombres `default` al ejecutar `helm list`, y podrá modificarlos y eliminarlos.

Los privilegios pueden gestionarse de forma granular con RLS, y puede ser de interés restringir el acceso según las diferentes columnas de la tabla:
* key
* type
* body
* name
* namespace
* version
* status
* owner
* createdAt
* modifiedAt 
