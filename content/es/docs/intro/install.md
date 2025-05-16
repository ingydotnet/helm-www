---
title: "Instalando Helm"
description: "Aprende cómo instalar y comenzar a usar Helm."
weight: 2
aliases: ["/docs/install/"]
---

Esta guía muestra cómo instalar el CLI de Helm. Helm se puede instalar desde el
código fuente o desde versiones binarias precompiladas.

## Desde el Proyecto Helm

El proyecto Helm proporciona dos formas de obtener e instalar Helm. Estos son los
métodos oficiales para obtener las versiones de Helm. Además de eso, la comunidad
de Helm proporciona métodos para instalar Helm a través de diferentes gestores de
paquetes. La instalación a través de esos métodos se puede encontrar debajo de
los métodos oficiales.

### Desde las Versiones Binarias

Cada [lanzamiento](https://github.com/helm/helm/releases) de Helm proporciona
versiones binarias para una variedad de sistemas operativos. Estas versiones
binarias se pueden descargar e instalar manualmente.

1. Descarga tu [versión deseada](https://github.com/helm/helm/releases)
2. Descomprímela (`tar -zxvf helm-v3.0.0-linux-amd64.tar.gz`)
3. Encuentra el binario `helm` en el directorio descomprimido y muévelo a su
   destino deseado (`mv linux-amd64/helm /usr/local/bin/helm`)

Desde allí, deberías poder ejecutar el cliente y [agregar el repositorio de
charts estable](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository):
`helm help`.

**Nota:** Las pruebas automatizadas de Helm se realizan solo para Linux AMD64
durante las compilaciones y lanzamientos de GitHub Actions. Las pruebas de otros
sistemas operativos son responsabilidad de la comunidad que solicita Helm para el
sistema operativo en cuestión.

### Desde Script

Helm ahora tiene un script de instalación que obtendrá automáticamente la última
versión de Helm y [la instalará localmente](https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3).

Puedes obtener ese script y luego ejecutarlo localmente. Está bien documentado
para que puedas leerlo y entender lo que hace antes de ejecutarlo.

```console
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Sí, puedes usar `curl
https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash` si
quieres vivir al límite.

## A través de Gestores de Paquetes

La comunidad de Helm proporciona la capacidad de instalar Helm a través de
gestores de paquetes del sistema operativo. Estos no están respaldados por el
proyecto Helm y no se consideran terceros confiables.

### Desde Homebrew (macOS)

Los miembros de la comunidad de Helm han contribuido con una fórmula de Helm
para Homebrew. Esta fórmula generalmente está actualizada.

```console
brew install helm
```

(Nota: También hay una fórmula para emacs-helm, que es un proyecto diferente.)

### Desde Chocolatey (Windows)

Los miembros de la comunidad de Helm han contribuido con un [paquete de
Helm](https://chocolatey.org/packages/kubernetes-helm) para
[Chocolatey](https://chocolatey.org/). Este paquete generalmente está actualizado.

```console
choco install kubernetes-helm
```

### Desde Scoop (Windows)

Los miembros de la comunidad de Helm han contribuido con un [paquete de
Helm](https://github.com/ScoopInstaller/Main/blob/master/bucket/helm.json) para
[Scoop](https://scoop.sh). Este paquete generalmente está actualizado.

```console
scoop install helm
```

### Desde Winget (Windows)

Los miembros de la comunidad de Helm han contribuido con un [paquete de
Helm](https://github.com/microsoft/winget-pkgs/tree/master/manifests/h/Helm/Helm)
para [Winget](https://learn.microsoft.com/en-us/windows/package-manager/). Este
paquete generalmente está actualizado.

```console
winget install Helm.Helm
```

### Desde Apt (Debian/Ubuntu)

Los miembros de la comunidad de Helm han contribuido con un [paquete de
Helm](https://helm.baltorepo.com/stable/debian/) para Apt. Este paquete
generalmente está actualizado.

```console
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Desde dnf/yum (fedora)
Desde Fedora 35, helm está disponible en el repositorio oficial.
Puedes instalar helm con:

```console
sudo dnf install helm
```

### Desde Snap

La comunidad [Snapcrafters](https://github.com/snapcrafters) mantiene la versión
Snap del [paquete Helm](https://snapcraft.io/helm):

```console
sudo snap install helm --classic
```

### Desde pkg (FreeBSD)

Los miembros de la comunidad de FreeBSD han contribuido con un [paquete de
Helm](https://www.freshports.org/sysutils/helm) para la [Colección de Puertos de
FreeBSD](https://man.freebsd.org/ports). Este paquete generalmente está
actualizado.

```console
pkg install helm
```

### Versiones de Desarrollo

Además de las versiones que puedes descargar o instalar, hay instantáneas de
desarrollo de Helm.

### Desde Versiones Canary

Las versiones "Canary" son versiones del software Helm que se construyen desde la
última rama `main`. No son versiones oficiales y pueden no ser estables. Sin
embargo, ofrecen la oportunidad de probar las características más recientes.

Los binarios Canary de Helm se almacenan en [get.helm.sh](https://get.helm.sh).
Aquí están los enlaces a las compilaciones comunes:

- [Linux AMD64](https://get.helm.sh/helm-canary-linux-amd64.tar.gz)
- [macOS AMD64](https://get.helm.sh/helm-canary-darwin-amd64.tar.gz)
- [Windows AMD64 Experimental](https://get.helm.sh/helm-canary-windows-amd64.zip)

### Desde el Código Fuente (Linux, macOS)

Compilar Helm desde el código fuente es un poco más de trabajo, pero es la mejor
manera de proceder si quieres probar la última versión (pre-lanzamiento) de Helm.

Debes tener un entorno Go funcionando.

```console
$ git clone https://github.com/helm/helm.git
$ cd helm
$ make
```

Si es necesario, obtendrá las dependencias y las almacenará en caché, y validará
la configuración. Luego compilará `helm` y lo colocará en `bin/helm`.

## Conclusión

En la mayoría de los casos, la instalación es tan simple como obtener un binario
`helm` precompilado. Este documento cubre casos adicionales para aquellos que
quieren hacer cosas más sofisticadas con Helm.

Una vez que hayas instalado exitosamente el Cliente Helm, puedes continuar usando
Helm para gestionar charts y [agregar el repositorio de charts
estable](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository).
