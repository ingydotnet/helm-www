---
title: "Guía de Plugins de Helm"
description: "Introduce cómo usar y crear plugins para extender la funcionalidad de Helm."
aliases: ["/docs/plugins/"]
weight: 12
---

Un plugin de Helm es una herramienta a la que se puede acceder a través de la CLI de `helm`, pero que no forma parte del código principal de Helm.

Puedes encontrar plugins existentes en la sección de [relacionados]({{< ref "related.md#helm-plugins" >}}) o buscando en [GitHub](https://github.com/search?q=topic%3Ahelm-plugin&type=Repositories).

Esta guía explica cómo usar y crear plugins.

## Visión general

Los plugins de Helm son herramientas adicionales que se integran perfectamente con Helm. Proporcionan una forma de extender el conjunto de funciones principales de Helm, sin requerir que cada nueva función se escriba en Go y se agregue a la herramienta principal.

Los plugins de Helm tienen las siguientes características:

- Se pueden agregar y eliminar de una instalación de Helm sin afectar la herramienta principal.
- Se pueden escribir en cualquier lenguaje de programación.
- Se integran con Helm y aparecerán en `helm help` y otros lugares.

Los plugins de Helm viven en `$HELM_PLUGINS`. Puedes encontrar el valor actual de esto, incluido el valor predeterminado cuando no está configurado en el entorno, usando el comando `helm env`.

El modelo de plugins de Helm está parcialmente basado en el modelo de plugins de Git. Por ello, a veces puedes escuchar que se refiere a `helm` como la capa _porcelana_, y a los plugins como la _fontanería_. Esto es una forma abreviada de sugerir que Helm proporciona la experiencia de usuario y la lógica de procesamiento de alto nivel, mientras que los plugins hacen el "trabajo detallado" de realizar una acción deseada.

## Instalando un plugin

Los plugins se instalan usando el comando `$ helm plugin install <ruta|url>`. Puedes pasar una ruta a un plugin en tu sistema de archivos local o una url de un repositorio VCS remoto. El comando `helm plugin install` clona o copia el plugin en la ruta/url dada dentro de `$HELM_PLUGINS`. Si instalas desde un VCS puedes especificar la versión con el argumento `--version`.

```console
$ helm plugin install https://github.com/adamreese/helm-env
```

Si tienes una distribución tar de un plugin, simplemente descomprime el plugin en el directorio `$HELM_PLUGINS`. También puedes instalar plugins en tarball directamente desde una url usando `helm plugin install https://dominio/ruta/al/plugin.tar.gz`

## Estructura de archivos del plugin

En muchos aspectos, un plugin es similar a un chart. Cada plugin tiene un directorio de nivel superior que contiene un archivo `plugin.yaml`. Pueden estar presentes archivos adicionales, pero solo se requiere el archivo `plugin.yaml`.

```console
$HELM_PLUGINS/
  |- last/
      |- plugin.yaml
```

## El archivo plugin.yaml

El archivo plugin.yaml es obligatorio para un plugin. Contiene los siguientes campos:

```yaml
name: El nombre del plugin (OBLIGATORIO)
version: Una versión SemVer 2 (OBLIGATORIO)
usage: Texto de uso en una línea mostrado en la ayuda
description: Descripción larga mostrada en lugares como helm help
ignoreFlags: Ignora flags pasados desde Helm
platformCommand: # Configura el comando a ejecutar según la plataforma
  - os: Coincidencia de SO, puede estar vacío u omitido para coincidir con todos los SO
    arch: Coincidencia de arquitectura, puede estar vacío u omitido para coincidir con todas las arquitecturas
    command: Comando del plugin a ejecutar
    args: Argumentos del comando del plugin
command: (OBSOLETO) Comando del plugin, usa platformCommand en su lugar
platformHooks: # Configura hooks de ciclo de vida del plugin según la plataforma
  install: # Comandos de instalación
    - os: Coincidencia de SO, puede estar vacío u omitido para coincidir con todos los SO
      arch: Coincidencia de arquitectura, puede estar vacío u omitido para coincidir con todas las arquitecturas
      command: Comando de instalación del plugin a ejecutar
      args: Argumentos del comando de instalación del plugin
  update: # Comandos de actualización
    - os: Coincidencia de SO, puede estar vacío u omitido para coincidir con todos los SO
      arch: Coincidencia de arquitectura, puede estar vacío u omitido para coincidir con todas las arquitecturas
      command: Comando de actualización del plugin a ejecutar
      args: Argumentos del comando de actualización del plugin
  delete: # Comandos de eliminación
    - os: Coincidencia de SO, puede estar vacío u omitido para coincidir con todos los SO
      arch: Coincidencia de arquitectura, puede estar vacío u omitido para coincidir con todas las arquitecturas
      command: Comando de eliminación del plugin a ejecutar
      args: Argumentos del comando de eliminación del plugin
hooks: # (Obsoleto) Hooks de ciclo de vida del plugin, usa platformHooks en su lugar
  install: Comando para instalar el plugin
  update: Comando para actualizar el plugin
  delete: Comando para eliminar el plugin
downloaders: # Configura la capacidad de downloaders
  - command: Comando a invocar
    protocols:
      - Esquema de protocolo soportado
```

### El campo `name`

`name` es el nombre del plugin. Cuando Helm ejecuta este plugin, este es el nombre que usará (por ejemplo, `helm NOMBRE` invocará este plugin).

_`name` debe coincidir con el nombre del directorio._ En el ejemplo anterior, eso significa que el plugin con `name: last` debe estar contenido en un directorio llamado `last`.

Restricciones sobre `name`:

- `name` no puede duplicar uno de los comandos de nivel superior existentes de `helm`.
- `name` debe limitarse a los caracteres ASCII a-z, A-Z, 0-9, `_` y `-`.

### El campo `version`

`version` es la versión SemVer 2 del plugin. `usage` y `description` se usan para generar el texto de ayuda de un comando.

### El campo `ignoreFlags`

El switch `ignoreFlags` le dice a Helm que _no_ pase flags al plugin. Así que si un plugin se llama con `helm myplugin --foo` y `ignoreFlags: true`, entonces `--foo` se descarta silenciosamente.

### El campo `platformCommand`

`platformCommand` configura el comando que el plugin ejecutará cuando sea llamado. No puedes establecer ambos, `platformCommand` y `command`, ya que esto resultará en un error. Se aplicarán las siguientes reglas para decidir qué comando usar:

- Si `platformCommand` está presente, se usará.
  - Si tanto `os` como `arch` coinciden con la plataforma actual, la búsqueda se detiene y se usará el comando.
  - Si `os` coincide y `arch` está vacío, se usará el comando.
  - Si ambos, `os` y `arch`, están vacíos, se usará el comando.
  - Si no hay coincidencia, Helm saldrá con un error.
- Si `platformCommand` no está presente y el obsoleto `command` está presente, se usará.
  - Si el comando está vacío, Helm saldrá con un error.

### El campo `platformHooks`

`platformHooks` configura los comandos que el plugin ejecutará para eventos de ciclo de vida. No puedes establecer ambos, `platformHooks` y `hooks`, ya que esto resultará en un error. Se aplicarán las siguientes reglas para decidir qué comando de hook usar:

- Si `platformHooks` está presente, se usará y se procesarán los comandos para el evento de ciclo de vida.
  - Si tanto `os` como `arch` coinciden con la plataforma actual, la búsqueda se detiene y se usará el comando.
  - Si `os` coincide y `arch` está vacío, se usará el comando.
  - Si ambos, `os` y `arch`, están vacíos, se usará el comando.
  - Si no hay coincidencia, Helm omitirá el evento.
- Si `platformHooks` no está presente y el obsoleto `hooks` está presente, se usará el comando para el evento de ciclo de vida.
  - Si el comando está vacío, Helm omitirá el evento.

## Construyendo un plugin

Aquí está el YAML de un plugin simple que ayuda a obtener el nombre del último release:

```yaml
name: last
version: 0.1.0
usage: obtener el nombre del último release
description: obtener el nombre del último release
ignoreFlags: false
platformCommand:
  - command: ${HELM_BIN}
    args:
      - list
      - --short
      - --max=1
      - --date
      - -r
```

Los plugins pueden requerir scripts y ejecutables adicionales. Los scripts pueden incluirse en el directorio del plugin y los ejecutables pueden descargarse mediante un hook. El siguiente es un ejemplo de plugin:

```console
$HELM_PLUGINS/
  |- myplugin/
    |- scripts/
      |- install.ps1
      |- install.sh
    |- plugin.yaml
```

```yaml
name: myplugin
version: 0.1.0
usage: plugin de ejemplo
description: plugin de ejemplo
ignoreFlags: false
platformCommand:
  - command: ${HELM_PLUGIN_DIR}/bin/myplugin
  - os: windows
    command: ${HELM_PLUGIN_DIR}\bin\myplugin.exe
platformHooks:
  install:
    - command: ${HELM_PLUGIN_DIR}/scripts/install.sh
    - os: windows
      command: pwsh
      args:
        - -c
        - ${HELM_PLUGIN_DIR}\scripts\install.ps1
  update:
    - command: ${HELM_PLUGIN_DIR}/scripts/install.sh
      args:
        - -u
    - os: windows
      command: pwsh
      args:
        - -c
        - ${HELM_PLUGIN_DIR}\scripts\install.ps1
        - -Update
```

Las variables de entorno se interpolan antes de que se ejecute el plugin. El patrón anterior ilustra la forma preferida de indicar dónde vive el programa del plugin.

### Comandos del plugin

Hay algunas estrategias para trabajar con los comandos de los plugins:

- Si un plugin incluye un ejecutable, el ejecutable para un `platformCommand:` debe estar empaquetado en el directorio del plugin o instalarse mediante un hook.
- La línea `platformCommand:` o `command:` expandirá cualquier variable de entorno antes de la ejecución. `$HELM_PLUGIN_DIR` apuntará al directorio del plugin.
- El comando en sí no se ejecuta en un shell. Así que no puedes poner un script de shell en una sola línea.
- Helm inyecta mucha configuración en variables de entorno. Echa un vistazo al entorno para ver qué información está disponible.
- Helm no hace suposiciones sobre el lenguaje del plugin. Puedes escribirlo en el que prefieras.
- Los comandos son responsables de implementar texto de ayuda específico para `-h` y `--help`. Helm usará `usage` y `description` para `helm help` y `helm help myplugin`, pero no manejará `helm myplugin --help`.

### Probando un plugin local

Primero necesitas encontrar tu ruta de `HELM_PLUGINS`. Para hacerlo, ejecuta el siguiente comando:

```bash
helm env
```

Cambia tu directorio actual al directorio que `HELM_PLUGINS` indica.

Ahora puedes agregar un enlace simbólico a la salida de tu build del plugin. En este ejemplo lo hicimos para `mapkubeapis`.

```bash
ln -s ~/GitHub/helm-mapkubeapis ./helm-mapkubeapis
```

## Plugins de descargador (Downloader)

Por defecto, Helm puede descargar Charts usando HTTP/S. Desde Helm 2.4.0, los plugins pueden tener una capacidad especial para descargar Charts desde fuentes arbitrarias.

Los plugins deben declarar esta capacidad especial en el archivo `plugin.yaml` (nivel superior):

```yaml
downloaders:
- command: "bin/mydownloader"
  protocols:
  - "myprotocol"
  - "myprotocols"
```

Si tal plugin está instalado, Helm puede interactuar con el repositorio usando el esquema de protocolo especificado invocando el `command`. El repositorio especial debe agregarse de manera similar a los regulares: `helm repo add favorito myprotocol://ejemplo.com/` Las reglas para los repositorios especiales son las mismas que para los regulares: Helm debe poder descargar el archivo `index.yaml` para descubrir y almacenar en caché la lista de Charts disponibles.

El comando definido se invocará con el siguiente esquema: `command certFile keyFile caFile full-URL`. Las credenciales SSL provienen de la definición del repo, almacenadas en `$HELM_REPOSITORY_CONFIG` (es decir, `$HELM_CONFIG_HOME/repositories.yaml`). Un plugin Downloader debe volcar el contenido bruto a stdout y reportar errores en stderr.

El comando downloader también soporta subcomandos o argumentos, permitiendo especificar por ejemplo `bin/mydownloader subcommand -d` en el `plugin.yaml`. Esto es útil si deseas usar el mismo ejecutable para el comando principal del plugin y el comando downloader, pero con un subcomando diferente para cada uno.

## Variables de entorno

Cuando Helm ejecuta un plugin, pasa el entorno externo al plugin y también inyecta algunas variables de entorno adicionales.

Variables como `KUBECONFIG` se establecen para el plugin si están configuradas en el entorno externo.

Las siguientes variables están garantizadas:

- `HELM_PLUGINS`: La ruta al directorio de plugins.
- `HELM_PLUGIN_NAME`: El nombre del plugin, como lo invoca `helm`. Así, `helm myplug` tendrá el nombre corto `myplug`.
- `HELM_PLUGIN_DIR`: El directorio que contiene el plugin.
- `HELM_BIN`: La ruta al comando `helm` (como lo ejecuta el usuario).
- `HELM_DEBUG`: Indica si la flag de depuración fue establecida por helm.
- `HELM_REGISTRY_CONFIG`: La ubicación para la configuración del registro (si se usa). Nota que el uso de Helm con registros es una característica experimental.
- `HELM_REPOSITORY_CACHE`: La ruta a los archivos de caché del repositorio.
- `HELM_REPOSITORY_CONFIG`: La ruta al archivo de configuración del repositorio.
- `HELM_NAMESPACE`: El namespace dado al comando `helm` (generalmente usando la flag `-n`).
- `HELM_KUBECONTEXT`: El nombre del contexto de configuración de Kubernetes dado al comando `helm`.

Adicionalmente, si se especificó explícitamente un archivo de configuración de Kubernetes, se establecerá como la variable `KUBECONFIG`.

## Nota sobre el análisis de flags

Al ejecutar un plugin, Helm analizará las flags globales para su propio uso. Ninguna de estas flags se pasa al plugin.
- `--burst-limit`: Se convierte en `$HELM_BURST_LIMIT`
- `--debug`: Si se especifica, `$HELM_DEBUG` se establece en `1`
- `--kube-apiserver`: Se convierte en `$HELM_KUBEAPISERVER`
- `--kube-as-group`: Se convierten en `$HELM_KUBEASGROUPS`
- `--kube-as-user`: Se convierte en `$HELM_KUBEASUSER`
- `--kube-ca-file`: Se convierte en `$HELM_KUBECAFILE`
- `--kube-context`: Se convierte en `$HELM_KUBECONTEXT`
- `--kube-insecure-skip-tls-verify`: Se convierte en `$HELM_KUBEINSECURE_SKIP_TLS_VERIFY`
- `--kube-tls-server-name`: Se convierte en `$HELM_KUBETLS_SERVER_NAME`
- `--kube-token`: Se convierte en `$HELM_KUBETOKEN`
- `--kubeconfig`: Se convierte en `$KUBECONFIG`
- `--namespace` y `-n`: Se convierte en `$HELM_NAMESPACE`
- `--qps`: Se convierte en `$HELM_QPS`
- `--registry-config`: Se convierte en `$HELM_REGISTRY_CONFIG`
- `--repository-cache`: Se convierte en `$HELM_REPOSITORY_CACHE`
- `--repository-config`: Se convierte en `$HELM_REPOSITORY_CONFIG`

Los plugins _deberían_ mostrar el texto de ayuda y luego salir para `-h` y `--help`. En todos los demás casos, los plugins pueden usar flags según corresponda.

## Proporcionando autocompletado de shell

Desde Helm 3.2, un plugin puede opcionalmente proporcionar soporte para autocompletado de shell como parte del mecanismo de autocompletado existente de Helm.

### Autocompletado estático

Si un plugin proporciona sus propias flags y/o subcomandos, puede informar a Helm de ellos teniendo un archivo `completion.yaml` ubicado en el directorio raíz del plugin. El archivo `completion.yaml` tiene la forma:

```yaml
name: <pluginName>
flags:
- <flag 1>
- <flag 2>
validArgs:
- <arg value 1>
- <arg value 2>
commands:
  name: <commandName>
  flags:
  - <flag 1>
  - <flag 2>
  validArgs:
  - <arg value 1>
  - <arg value 2>
  commands:
     <y así sucesivamente, recursivamente>
```

Notas:

1. Todas las secciones son opcionales pero deben proporcionarse si corresponde.
1. Las flags no deben incluir el prefijo `-` o `--`.
1. Se pueden y deben especificar tanto flags cortas como largas. Una flag corta no necesita estar asociada con su forma larga correspondiente, pero ambas formas deben estar listadas.
1. Las flags no necesitan estar ordenadas de ninguna manera, pero deben listarse en el punto correcto de la jerarquía de subcomandos del archivo.
1. Las flags globales existentes de Helm ya son manejadas por el mecanismo de autocompletado de Helm, por lo tanto los plugins no necesitan especificar las siguientes flags `--debug`, `--namespace` o `-n`, `--kube-context`, y `--kubeconfig`, o cualquier otra flag global.
1. La lista `validArgs` proporciona una lista estática de posibles completados para el primer parámetro después de un subcomando. No siempre es posible proporcionar tal lista de antemano (ver la sección [Autocompletado dinámico](#dynamic-completion)), en cuyo caso se puede omitir la sección `validArgs`.

El archivo `completion.yaml` es completamente opcional. Si no se proporciona, Helm simplemente no proporcionará autocompletado de shell para el plugin (a menos que el plugin soporte [Autocompletado dinámico](#dynamic-completion)). Además, agregar un archivo `completion.yaml` es retrocompatible y no afectará el comportamiento del plugin al usar versiones antiguas de helm.

Como ejemplo, para el [`plugin fullstatus`](https://github.com/marckhouzam/helm-fullstatus) que no tiene subcomandos pero acepta las mismas flags que el comando `helm status`, el archivo `completion.yaml` es:

```yaml
name: fullstatus
flags:
- o
- output
- revision
```

Un ejemplo más intrincado para el [`plugin 2to3`](https://github.com/helm/helm-2to3), tiene un archivo `completion.yaml` de:

```yaml
name: 2to3
commands:
- name: cleanup
  flags:
  - config-cleanup
  - dry-run
  - l
  - label
  - release-cleanup
  - s
  - release-storage
  - tiller-cleanup
  - t
  - tiller-ns
  - tiller-out-cluster
- name: convert
  flags:
  - delete-v2-releases
  - dry-run
  - l
  - label
  - s
  - release-storage
  - release-versions-max
  - t
  - tiller-ns
  - tiller-out-cluster
- name: move
  commands:
  - name: config
    flags:
    - dry-run
```

### Autocompletado dinámico

También desde Helm 3.2, los plugins pueden proporcionar su propio autocompletado de shell dinámico. El autocompletado dinámico es el completado de valores de parámetros o flags que no se pueden definir de antemano. Por ejemplo, el completado de los nombres de releases de helm disponibles actualmente en el clúster.

Para que el plugin soporte autocompletado dinámico, debe proporcionar un archivo **ejecutable** llamado `plugin.complete` en su directorio raíz. Cuando el script de autocompletado de Helm requiera completados dinámicos para el plugin, ejecutará el archivo `plugin.complete`, pasándole la línea de comandos que necesita completarse. El ejecutable `plugin.complete` deberá tener la lógica para determinar cuáles son las opciones de completado adecuadas y mostrarlas por salida estándar para que el script de autocompletado de Helm las consuma.

El archivo `plugin.complete` es completamente opcional. Si no se proporciona, Helm simplemente no proporcionará autocompletado dinámico para el plugin. Además, agregar un archivo `plugin.complete` es retrocompatible y no afectará el comportamiento del plugin al usar versiones antiguas de helm.

La salida del script `plugin.complete` debe ser una lista separada por saltos de línea como:

```console
rel1
rel2
rel3
```

Cuando se llama a `plugin.complete`, el entorno del plugin se configura igual que cuando se llama al script principal del plugin. Por lo tanto, las variables `$HELM_NAMESPACE`, `$HELM_KUBECONTEXT` y todas las demás variables del plugin ya estarán configuradas, y sus flags globales correspondientes serán eliminadas.

El archivo `plugin.complete` puede ser de cualquier forma ejecutable; puede ser un script de shell, un programa Go, o cualquier otro tipo de programa que Helm pueda ejecutar. El archivo `plugin.complete` ***debe*** tener permisos de ejecución para el usuario. El archivo `plugin.complete` ***debe*** salir con un código de éxito (valor 0).

En algunos casos, el autocompletado dinámico requerirá obtener información del clúster de Kubernetes. Por ejemplo, el plugin `helm fullstatus` requiere un nombre de release como entrada. En el plugin `fullstatus`, para que su script `plugin.complete` proporcione completado para los nombres de release actuales, simplemente puede ejecutar `helm list -q` y mostrar el resultado.

Si se desea usar el mismo ejecutable para la ejecución del plugin y para el completado, el script `plugin.complete` puede llamar al ejecutable principal del plugin con algún parámetro o flag especial; cuando el ejecutable principal del plugin detecte el parámetro o flag especial, sabrá que debe ejecutar el completado. En nuestro ejemplo, `plugin.complete` podría implementarse así:

```sh
#!/usr/bin/env sh

# "$@" es toda la línea de comandos que requiere completado.
# Es importante poner comillas dobles a la variable "$@" para preservar un posible último parámetro vacío.
$HELM_PLUGIN_DIR/status.sh --complete "$@"
```

El script real del plugin `fullstatus` (`status.sh`) debe entonces buscar la flag `--complete` y, si la encuentra, mostrar los completados adecuados.

### Consejos y trucos

1. El shell filtrará automáticamente las opciones de completado que no coincidan con la entrada del usuario. Por lo tanto, un plugin puede devolver todos los completados relevantes sin eliminar los que no coincidan con la entrada del usuario. Por ejemplo, si la línea de comandos es `helm fullstatus ngin<TAB>`, el script `plugin.complete` puede mostrar *todos* los nombres de release (del namespace `default`), no solo los que comienzan con `ngin`; el shell solo retendrá los que comiencen con `ngin`.
1. Para simplificar el soporte de autocompletado dinámico, especialmente si tienes un plugin complejo, puedes hacer que tu script `plugin.complete` llame a tu script principal del plugin y solicite las opciones de completado. Consulta la sección [Autocompletado dinámico](#dynamic-completion) anterior para un ejemplo.
1. Para depurar el autocompletado dinámico y el archivo `plugin.complete`, se puede ejecutar lo siguiente para ver los resultados del completado:
    - `helm __complete <pluginName> <argumentos a completar>`. Por ejemplo:
    - `helm __complete fullstatus --output js<ENTER>`,
    - `helm __complete fullstatus -o json ""<ENTER>`


</rewritten_file> 
