# Proxmox Scripts

Conjunto de scripts para automatizar tareas en Proxmox **PVE 7** _(Proxmox Virtual Environment 7)_

## Proxmox Tools

- [Post instalación Proxmox VE7](#post-instalación-proxmox-ve7)
- [Escalado de CPU](#escalado-de-cpu)
- [Actualización masiva de contenedores](#actualización-masiva-de-contenedores)
- [Tema oscuro en Proxmox](#tema-oscuro-en-proxmox)
- [Post instalación Proxmox Backap Server](#post-instalación-proxmox-backap-server)

## Sistemas operativos

- [Debian Stable](#debian-stable)
- [Ubuntu Core](#ubuntu-core)

## Servidores de base de datos

- [MariaDB LXC](#mariadb-lxc)
- [PostgreSQL LXC](#postgresql-lxc)

## Servidores de archivos

- [NodeJS](#nodejs-lxc)
- [NextCloud](#nextcloudpi-lxc)
- [RouterOS Mikrotik](#routeros-vm-de-mikrotik)

## Herramientas

- [Docker en LXC](#docker-lxc)
- [Navegador de archivos](#navegador-de-archivos)
- [Servidor de desarrollo con VSCode](#servidor-de-desarrollo-con-vscode)
- [Administración del sistema con Webmin](#administración-del-sistema-con-webmin)
- [Sincronización de archivos](#sincronización-de-archivos)

## Proxmox Tools

### Post instalación Proxmox VE7

**PVE7 Post Install** permite deshabilitar Enterprise Repo, agregar/corregir fuentes de PVE7, habilitar el repositorio sin suscripción, agregar repositorios Beta, deshabilitar suscripción Nag, actualizar Proxmox VE y reiniciar PVE.

Ejecute lo siguiente en una shell de Proxmox. ⚠️ SOLO PVE7

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/post-pve-install.sh)"

Se recomienda responder `y` a todas las opciones.

### Escalado de CPU

**CPU Scaling Governor** permite que el sistema operativo escale la frecuencia de la CPU hacia arriba o hacia abajo para ahorrar energía o mejorar el rendimiento.

Ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/scaling-governor.sh)"

### Actualización masiva de contenedores

**Update LXC** actualiza todos los LXC de forma rápida y sencilla (Ubuntu, Debian, AlpineLinux, etc.)

Ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/update-lxcs.sh)"

### Tema oscuro en Proxmox

Un tema oscuro para la interfaz de usuario web de Proxmox.

Ejecute lo siguiente en una shell de Proxmox.

    bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install

Para desinstalar el tema, simplemente ejecute el script con el comando `uninstall`.

### Post instalación Proxmox Backap Server

**PBS Post Install** permite deshabilitar el Enterprise Repo, agregar/corregir fuentes de PBS, habilitar el repositorio sin suscripción, agregar repositorio Beta, deshabilitar suscripción Nag, actualizar el servidor de copia de seguridad de Proxmox y reiniciar PBS.

Ejecute lo siguiente en una shell de Proxmox. ⚠️ Proxmox Backup Server SOLAMENTE

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/post-pbs-install.sh)"

Se recomienda responder `y` a todas las opciones.

## Sistemas operativos

### Debian Stable

Para crear un LXC con Debian (curl & sudo), ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/debian.sh)"

⚡ Default Settings: 512MiB RAM - 2GB Storage - 1vCPU ⚡

⚙️ Para actualizar Debian

Ejecute dentro de la consola del contenedor: *apt update && apt upgrade -y*

### Ubuntu Core

Para crear LXC con Ubuntu, ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/ubuntu.sh)"

⚡ Default Settings: 512MiB RAM - 2GB Storage - 1vCPU - 22.04 ⚡

⚙️ Para actualizar Ubuntu

Ejecute dentro de la consola del contenedor: *apt update && apt upgrade -y*

## Servidores de base de datos

### Mariadb LXC

Para crear un LXC con Mariadb, ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/mariadb.sh)"

Configuración predeterminada: 1GB de RAM - 4GB Almacenamiento - 1vCPU

Para permitir a MariaDB escuchar conexiones remotas, necesita editar el archivo predeterminado. Para ello, abra la consola en su MariaDB lxc:

    nano /etc/mysql/my.cnf

Descomenta `port = 3306`, guardar y salir del editor con "Ctrl-O", "Enter" y "Ctrl X". Luego:

    nano /etc/mysql/mariadb.conf.d/50-server.cnf

Comente `bind-address = 127.0.0.1`, guardar y salir del editor con "Ctrl-O", "Enter" y "Ctrl X".

Para las nuevas instalaciones de MariaDB, el siguiente paso es ejecutar el script de seguridad incluido. Este script cambia algunas de las opciones predeterminadas menos seguras. Lo usaremos para bloquear los logins remotos y para eliminar usuarios de bases de datos no utilizados.

Ejecute lo siguiente:

    sudo mysql_secure_installation

Y en el script interactivo debería contestar lo siguiente:

    Enter current password for root (enter for none): enter
    Switch to unix_socket authentication [Y/n] y
    Change the root password? [Y/n] n
    Remove anonymous users? [Y/n] y
    Disallow root login remotely? [Y/n] y
    Remove test database and access to it? [Y/n] y
    Reload privilege tables now? [Y/n] y

Crearemos una nueva cuenta llamada admin con las mismas capacidades que la cuenta raíz, pero configurada para la autenticación de contraseñas.

    sudo mariadb

Creamos un nuevo administrador (cambie el nombre de usuario y la contraseña para que coincidan con sus preferencias)

En contenedores administrados por Proxmox no tiene sentido el acceso externo. Una cuenta local es sufciente y garantiza la seguridad del contenedor

    CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';

Le damos privilegios de root (cambie el nombre de usuario y la contraseña para que coincidan con sus preferencias)

    GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;

Puede permitir el acceso root desde cualquier lugar de su red de área local (LAN), con direcciones en la subred 192.168.100.0/24. Cambie el nombre de usuario, la contraseña y la subred para que coincida con tus preferencias:

    GRANT ALL ON *.* TO 'admin'@'192.168.100.%' IDENTIFIED BY 'password' WITH GRANT OPTION;

Refrescar los privilegios para asegurar que se guarden y estén disponibles en la sesión actual:

    FLUSH PRIVILEGES;

Después de esto, salga de la consola de MariaDB:

    exit

Ya puede iniciar sesión como el nuevo usuario que acaba de crear:

    mysql -u admin -p

Reiniciar el lxc

Puede comprobar el estado del servicio con:

    sudo systemctl status mariadb

Para actualizar Mariadb, dentro de la consola ejecute:

    apt update && apt upgrade -y

### PostgreSQL LXC

PostgreSQL LXC
Opción para instalar Adminer

PostgreSQL, también conocido como Postgres, es un sistema de gestión de bases de datos relacionales de código libre y de código abierto que enfatiza la extensibilidad y el cumplimiento SQL.

Para crear un nuevo Proxmox PostgreSQL LXC, ejecute el siguiente en la Concha Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/postgresql-v4.sh)"

Configuración predeterminada: 1GB de RAM - 4GB Almacenamiento - 1vCPU

Para asegurarse de que nuestro PostgreSQL está asegurado con una contraseña fuerte, establezca una contraseña para su usuario de su sistema y luego cambie la cuenta de usuario de usuario de la base de datos predeterminada

Cambiar contraseña de usuario

passwd postgres

Inicia sesión usando la cuenta del sistema Postgres

su - postgres

Ahora, cambie la contraseña de la base de datos Administrador

psql -c "ALTER USER postgres WITH PASSWORD 'your-password';"

Crear un nuevo usuario.

psql

CREATE USER admin WITH PASSWORD 'your-password';

Crear una nueva base de datos:

CREATE DATABASE homeassistant;

Confiera todos los derechos o privilegios en la base de datos creada al usuario

GRANT ALL ON DATABASE homeassistant TO admin;

A la salida psql

\q

Luego tipo salida para volver a la raíz

Cambiar la grabadora: db_url:en su configuración de HA.yaml

Ejemplo:

recorder:
db_url: postgresql://admin:your-password@192.168.100.20:5432/homeassistant?client_encoding=utf8

Actualizar PostgreSQL

Corre en la consola LXC

apt update && apt upgrade -y

Adminer (an pasado phpMinAdmin) es una herramienta de gestión de bases de datos completa

Interfaz de administración: IP/adminor/

## Servidores de archivos

### NodeJS LXC

Esto instalará NodeJS 16.x, Nginx como proxy reveso, Node PM2 para manejar los procesos al inicio y Nodemo para monitorear cambios

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/nodejs.sh)"

todo: crear u bloque de servidor en nginx, instalar certbot

### NextCloudPi LXC

NextCloudPi LXC es la solución de colaboración auto-anfitriada más popular.

Para crear un nuevo Proxmox NextCloudPi LXC, ejecute el siguiente en la Concha Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/nextcloudpi-v4.sh)"

Configuración predeterminada: 2GB de RAM - 8GB Almacenamiento - 2vCPU

Establecer dominios de confianza nc

Corre en la consola LXC

sudo ncp-config

Ir a config . . . . . . . . . . . . . . . . . . . . . . . . . . . . 0.0.0.0o una IP estática NextCloudPi

Vuelva al aviso de comando y reinicie Apache2 sudo service apache2 restart

NextCloudPi Interface: (https)IP/

### RouterOS VM de Mikrotik

Mikrotik RouterOS se puede instalar en un PC y lo convertirá en un router con todas las características necesarias: enrutación, cortafuegas, gestión de ancho de banda, punto de acceso inalámbrico, enlace de backhaul, pasarela de puntos calientes, servidor VPN y más.

Para crear un nuevo Proxmox Mikrotik RouterOS VM, ejecute el siguiente en la Concha Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/vm/mikrotik-routeros-v4.sh)"

Configuración por defecto: 1GB de RAM - 2GB Almacenamiento - 1CPU

La configuración se realiza a través de la consola VM.

## Herramientas

### Docker LXC

Options to Install Portainer and/or Docker Compose V2

Docker is an open-source project for automating the deployment of applications as portable, self-sufficient containers.

Para crear LXC con Docker LXC, ejecute lo siguiente en una shell de Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/docker-v4.sh)"

⚡ Default Settings: 2GB RAM - 4GB Storage - 2vCPU ⚡

⚠ Run Compose V2 by replacing the hyphen (-) with a space, using docker compose, instead of docker-compose.

Portainer Interface: IP:9000

⚙️ To Update

Ejecute dentro de la consola del LXC

apt update && apt upgrade -y

### Navegador de archivos

File Browser es un software de creación-su-prop-prop-bud--cuber-quete donde pueda instalarlo en un servidor, dirigirlo a una ruta y luego acceder a tus archivos a través de una bonita interfaz web. Muchas características disponibles.

Para instalar el navegador de archivos, ejecute el siguiente en la consola LXC.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/filebrowser.sh)"

Interfaz del navegador de archivos: IP:8080

Initrimonio de inicio

Nombre de usuario admin

contraseña changeme

Para actualizar el navegador de archivos

curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash

### Servidor de desarrollo con VSCode

Servidor de código VS

VS Code Server es un servicio que puede ejecutar en una máquina de desarrollo remoto, como su PC de escritorio o una máquina virtual (VM). Le permite conectarse de forma segura a esa máquina remota desde cualquier lugar a través de una URL vscode.dev, sin el requisito de SSH.

Para instalar VS Code Server, ejecute el siguiente lugar en la consola LXC.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/code-server.sh)"

VS Code Server Interfaz: IP:8680

### Administración del sistema con Webmin

Administración del sistema de Webmin

Si prefiere administrar todos los aspectos de su Proxmox LXC desde una interfaz gráfica en lugar de la interfaz de la línea de comandos, Webmin podría ser adecuado para usted. Los beneficios incluyen actualizaciones automáticas diarias de seguridad, copias de seguridad y restauración, gestor de archivos con editor, panel de control web y monitoreo de sistemas preconfigurados con alertas de correo electrónico opcionales.

Para instalar Webmin System Administration (Screenshot), ejecute el siguiente en la consola LXC.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/webmin.sh)"

Webmin Interface: (httpsIP:10000

Inigertronate

Nombre de usuario root

contraseña root

Actualizar Webmin

Update from the Webmin UI

Para desinstalar Webmin

Corre en la consola LXC

bash /etc/webmin/uninstall.sh

### Sincronización de archivos

[Syncthing](https://syncthing.net/) es un programa de sincronización de archivos continuo. Sincroniza archivos entre dos o más ordenadores.

Para crear un nuevo LXC de sincronización de Proxmox, ejecute el siguiente comando en un shell de Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/syncthing-v4.sh)"

Configuración predeterminada: 2GB de RAM - 8GB Almacenamiento - 2vCPU

Para el inicializar, ejecute `syncthing --gui-address=0.0.0.0:8384` en la consola del LXC, luego vaya a su_lxc_ip:8384. En los ajustes configuramos la dirección IP LXC bajo la GUI (también configura la GUI Authentication User y GUI Authentication Password) y lapestaña Connections. Luego guarde y reinicie el LXC.

Interfaz de sincronización: su_ip:8384

Actualizar Syncthing

apt update && apt upgrade -y
