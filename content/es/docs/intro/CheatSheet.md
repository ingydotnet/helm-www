---
title: "Hoja de Referencia"
description: "Hoja de referencia de Helm"
weight: 4
---

Hoja de referencia de Helm con todos los comandos necesarios para gestionar una
aplicación a través de Helm.

-----------------------------------------------------------------------------------------------------------------------------------------------
### Interpretaciones/contexto básico

Chart:
- Es el nombre de tu chart en caso de que haya sido descargado y descomprimido.
- Es <nombre_repo>/<nombre_chart> en caso de que el repositorio haya sido
  agregado pero el chart no descargado.
- Es la URL/Ruta absoluta al chart.

Name:
- Es el nombre que quieres dar a tu instalación actual del chart de Helm.

Release:
- Es el nombre que asignaste a una instancia de instalación.

Revision:
- Es el valor del comando Helm history.

Repo-name:
- El nombre de un repositorio.

DIR:
- Nombre/ruta del directorio

------------------------------------------------------------------------------------------------------------------------------------------------

### Gestión de Charts

```bash
helm create <name>                      # Crea un directorio de chart junto con los archivos y directorios comunes usados en un chart.
helm package <chart-path>               # Empaqueta un chart en un archivo de archivo de chart versionado.
helm lint <chart>                       # Ejecuta pruebas para examinar un chart e identificar posibles problemas.
helm show all <chart>                   # Inspecciona un chart y lista su contenido.
helm show values <chart>                # Muestra el contenido del archivo values.yaml.
helm pull <chart>                       # Descarga/extrae chart.
helm pull <chart> --untar=true          # Si se establece como true, descomprimirá el chart después de descargarlo.
helm pull <chart> --verify              # Verifica el paquete antes de usarlo.
helm pull <chart> --version <number>    # Por defecto se usa la última versión, especifica una restricción de versión para la versión del chart a usar.
helm dependency list <chart>            # Muestra una lista de las dependencias de un chart.
```
--------------------------------------------------------------------------------------------------------------------------------------------------

### Instalar y Desinstalar Aplicaciones

```bash
helm install <name> <chart>                           # Instala el chart con un nombre.
helm install <name> <chart> --namespace <namespace>   # Instala el chart en un namespace específico.
helm install <name> <chart> --set key1=val1,key2=val2 # Establece valores en la línea de comandos (puedes especificar múltiples o separar valores con comas).
helm install <name> <chart> --values <yaml-file/url>  # Instala el chart con tus valores especificados.
helm install <name> <chart> --dry-run --debug         # Ejecuta una instalación de prueba para validar el chart.
helm install <name> <chart> --verify                  # Verifica el paquete antes de usarlo.
helm install <name> <chart> --dependency-update       # Actualiza las dependencias si faltan antes de instalar el chart.
helm uninstall <name>                                 # Desinstala un lanzamiento.
```
------------------------------------------------------------------------------------------------------------------------------------------------
### Realizar Actualización y Reversión de Aplicaciones

```bash
helm upgrade <release> <chart>                            # Actualiza un lanzamiento.
helm upgrade <release> <chart> --atomic                   # Si se establece, el proceso de actualización revierte los cambios realizados en caso de actualización fallida.
helm upgrade <release> <chart> --dependency-update        # Actualiza las dependencias si faltan antes de instalar el chart.
helm upgrade <release> <chart> --version <version_number> # Especifica una restricción de versión para la versión del chart a usar.
helm upgrade <release> <chart> --values                   # Especifica valores en un archivo YAML o una URL (puedes especificar múltiples).
helm upgrade <release> <chart> --set key1=val1,key2=val2  # Establece valores en la línea de comandos (puedes especificar múltiples o separar valores).
helm upgrade <release> <chart> --force                    # Fuerza actualizaciones de recursos a través de una estrategia de reemplazo.
helm rollback <release> <revision>                        # Revierte un lanzamiento a una revisión específica.
helm rollback <release> <revision>  --cleanup-on-fail     # Permite la eliminación de nuevos recursos creados en esta reversión cuando la reversión falla.
```
------------------------------------------------------------------------------------------------------------------------------------------------
### Listar, Agregar, Eliminar y Actualizar Repositorios

```bash
helm repo add <repo-name> <url>   # Agrega un repositorio desde internet.
helm repo list                    # Lista los repositorios de charts agregados.
helm repo update                  # Actualiza la información de charts disponibles localmente desde los repositorios de charts.
helm repo remove <repo_name>      # Elimina uno o más repositorios de charts.
helm repo index <DIR>             # Lee el directorio actual y genera un archivo de índice basado en los charts encontrados.
helm repo index <DIR> --merge     # Fusiona el índice generado con un archivo de índice existente.
helm search repo <keyword>        # Busca en los repositorios una palabra clave en los charts.
helm search hub <keyword>         # Busca charts en Artifact Hub o tu propia instancia de hub.
```
-------------------------------------------------------------------------------------------------------------------------------------------------
### Monitoreo de Lanzamientos de Helm

```bash
helm list                       # Lista todos los lanzamientos para un namespace especificado, usa el contexto de namespace actual si no se especifica namespace.
helm list --all                 # Muestra todos los lanzamientos sin ningún filtro aplicado, se puede usar -a.
helm list --all-namespaces      # Lista lanzamientos en todos los namespaces, podemos usar -A.
helm list -l key1=value1,key2=value2 # Selector (consulta de etiqueta) para filtrar, soporta '=', '==', y '!='.
helm list --date                # Ordena por fecha de lanzamiento.
helm list --deployed            # Muestra lanzamientos desplegados. Si no se especifica otro, esto se habilitará automáticamente.
helm list --pending             # Muestra lanzamientos pendientes.
helm list --failed              # Muestra lanzamientos fallidos.
helm list --uninstalled         # Muestra lanzamientos desinstalados (si se usó 'helm uninstall --keep-history').
helm list --superseded          # Muestra lanzamientos reemplazados.
helm list -o yaml               # Imprime la salida en el formato especificado. Valores permitidos: table, json, yaml (predeterminado table).
helm status <release>           # Este comando muestra el estado de un lanzamiento nombrado.
helm status <release> --revision <number>   # Si se establece, muestra el estado del lanzamiento nombrado con la revisión.
helm history <release>          # Revisiones históricas para un lanzamiento dado.
helm env                        # Env imprime toda la información del entorno en uso por Helm.
```
-------------------------------------------------------------------------------------------------------------------------------------------------
### Descargar Información de Lanzamientos

```bash
helm get all <release>      # Una colección legible de información sobre las notas, hooks, valores suministrados y archivo de manifiesto generado del lanzamiento dado.
helm get hooks <release>    # Este comando descarga hooks para un lanzamiento dado. Los hooks están formateados en YAML y separados por el separador YAML '---\n'.
helm get manifest <release> # Un manifiesto es una representación codificada en YAML de los recursos de Kubernetes que fueron generados desde el/los chart(s) de este lanzamiento. Si un chart depende de otros charts, esos recursos también se incluirán en el manifiesto.
helm get notes <release>    # Muestra las notas proporcionadas por el chart de un lanzamiento nombrado.
helm get values <release>   # Descarga un archivo de valores para un lanzamiento dado. Usa -o para formatear la salida.
```
-------------------------------------------------------------------------------------------------------------------------------------------------
### Gestión de Plugins

```bash
helm plugin install <path/url>      # Instala plugins.
helm plugin list                    # Muestra una lista de todos los plugins instalados.
helm plugin update <plugin>         # Actualiza plugins.
helm plugin uninstall <plugin>      # Desinstala un plugin.
```
-------------------------------------------------------------------------------------------------------------------------------------------------
