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

- [MariaDB](#mariadb-lxc)
- [PostgreSQL](#postgresql-lxc)

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
- [Consola Unifi](#consola-unifi)

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

⚡ Default Settings: 512MiB RAM - 2GB Storage - 1vCPU - 22.04 LTS ⚡

⚙️ Para actualizar Ubuntu

Ejecute dentro de la consola del contenedor: *apt update && apt upgrade -y*

## Servidores de base de datos

### Mariadb LXC

Para crear un LXC con Mariadb, ejecute lo siguiente en una shell de Proxmox.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/mariadb.sh)"

⚡ Default Settings: 1GB RAM - 4GB Storage - 1vCPU ⚡

Para permitir a MariaDB escuchar conexiones remotas, necesita editar el archivo de configuración predeterminado. En la shell del contenedor:

    nano /etc/mysql/my.cnf

Descomenta `port = 3306`, guardar y salir del editor con "Ctrl-O", "Enter" y "Ctrl X". Luego:

    nano /etc/mysql/mariadb.conf.d/50-server.cnf

Comente `bind-address = 127.0.0.1`, guardar y salir del editor con "Ctrl-O", "Enter" y "Ctrl X".

Para las nuevas instalaciones de MariaDB, el siguiente paso es ejecutar el script de seguridad incluido. Este script cambia algunas de las opciones predeterminadas menos seguras. Lo usaremos para bloquear los logins remotos y para eliminar usuarios de bases de datos no utilizados.

Ejecute lo siguiente:

    mysql_secure_installation

Y en el script interactivo debería contestar lo siguiente:

    Enter current password for root (enter for none): enter
    Switch to unix_socket authentication [Y/n] y
    Change the root password? [Y/n] n
    Remove anonymous users? [Y/n] y
    Disallow root login remotely? [Y/n] y
    Remove test database and access to it? [Y/n] y
    Reload privilege tables now? [Y/n] y

Crearemos una nueva cuenta llamada admin con las mismas capacidades que la cuenta raíz, pero configurada para la autenticación de contraseñas.

    mariadb

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

    systemctl status mariadb

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

### Nodejs LXC

Para crear un LXC con un servidor Node y Nginx como proxy inverso, ejecute lo siguiente en una shell de Proxmox:

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/nodejs.sh)"

Lo que sigue se ejecuta sobre una shell dentro del contenedor como root.

#### Crear un bloque de servidor en Nginx

Los bloques de servidor (similares a los hosts virtuales en Apache) se pueden usar para encapsular detalles de configuración y alojar más de un dominio desde un solo servidor.

Creamos el directorio para TU_DOMINIO de la siguiente manera, utilizando el flag para crear los directorios primarios necesarios: ``-p``

    mkdir -p /var/www/TU_DOMINIO/html

A continuación, asigne la propiedad del directorio con la variable de entorno:$USER

    chown -R $USER:$USER /var/www/TU_DOMINIO/html

Para permitir que el propietario lea, escriba y ejecute los archivos mientras concede solo permisos de lectura y ejecución a grupos y otros, puede ingresar el siguiente comando:

    chmod -R 755 /var/www/TU_DOMINIO

A continuación, cree una página de muestra utilizando o su editor favorito:

    nano /var/www/TU_DOMINIO/html/index.html

En el interior, agregue el siguiente ejemplo de HTML:

```html
<html>
    <head>
        <title>Bienvenido aTU_DOMINIO!</title>
    </head>
    <body>
        <h1>El bloque de servidor para TU_DOMINIO esta funcionando!</h1>
    </body>
</html>
```

Guarde y cierre el archivo.

Para que Nginx sirva este contenido, es necesario crear un bloque de servidor con las directivas correctas. En lugar de modificar el archivo de configuración predeterminado directamente, hagamos uno nuevo:

    nano /etc/nginx/sites-available/TU_DOMINIO

Pegue el siguiente bloque de configuración, que es similar al predeterminado, pero actualizado para nuestro nuevo directorio y nombre de dominio:

```nginx
server {
        listen 80;
        listen [::]:80;

        root /var/www/TU_DOMINIO/html;
        index index.html index.htm index.nginx-debian.html;

        server_name TU_DOMINIO www.TU_DOMINIO;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

A continuación, habilitemos el archivo creando un enlace desde él al directorio, que Nginx lee durante el inicio:

    ln -s /etc/nginx/sites-available/TU_DOMINIO /etc/nginx/sites-enabled/

Nota: Nginx utiliza una práctica común llamada enlaces simbólicos para rastrear cuáles de los bloques de su servidor están habilitados. Crear un enlace simbólico es como crear un acceso directo en el disco, de modo que luego pueda eliminar el acceso directo del directorio mientras mantiene el bloqueo del servidor.

Dos bloques de servidor ahora están habilitados y configurados para responder a las solicitudes basadas en sus directivas

- TU_DOMINIO: Responderá a las solicitudes de TU_DOMINIO y www.TU_DOMINIO
- default: responderá a cualquier solicitud en el puerto 80 que no coincida con los otros dos bloques.

Para evitar un posible problema que puede surgir al agregar nombres de servidor adicionales, es necesario ajustar un solo valor en el archivo de configuración de Nginx.

    nano /etc/nginx/nginx.conf

Busque la directiva ``server_names_hash_bucket_size`` y descoméntela

A continuación, pruebe para asegurarse de que no haya errores de sintaxis en ninguno de sus archivos Nginx:

    nginx -t

Si no hay ningún problema, reinicie Nginx para habilitar sus cambios:

    systemctl restart nginx

Nginx ahora debería estar sirviendo su nombre de dominio. Puede probar esto navegando a , donde debería ver algo como esto: http://TU_DOMINIO

#### Instalar certbot

Instale Certbot y su complemento Nginx con:

    apt install certbot python3-certbot-nginx

Certbot ahora está listo para usar, pero para que configure automáticamente SSL para Nginx, necesitamos verificar parte de la configuración de Nginx.

Certbot necesita poder encontrar el bloque correcto en su configuración de Nginx para que pueda configurar automáticamente SSL. Específicamente, lo hace buscando una directiva que coincida con el dominio para el que solicita un certificado.

    nano /etc/nginx/sites-available/TU_DOMINIO

Busque la línea ``server_name``

    /etc/nginx/sites-available/TU_DOMINIO

actualícelo para que coincida co su dominio. Guarde el archivo, salga del editor y compruebe la sintaxis de las ediciones de configuración:

    nginx -t

Si recibe un error, vuelva a abrir el archivo de bloqueo del servidor y compruebe si hay errores tipográficos o caracteres faltantes. Una vez que la sintaxis de su archivo de configuración sea correcta, vuelva a cargar Nginx para cargar la nueva configuración:

    systemctl reload nginx

Certbot proporciona una variedad de formas de obtener certificados SSL a través de complementos. El complemento Nginx se encargará de reconfigurar Nginx y volver a cargar la configuración cuando sea necesario. Para usar este complemento, escriba lo siguiente:

    certbot --nginx -d TU_DOMINIO -d www.TU_DOMINIO

Esto se ejecuta con el complemento, utilizando para especificar los nombres de dominio para los que nos gustaría que el certificado sea válido.

Si es la primera vez que se ejecuta, se le pedirá que introduzca una dirección de correo electrónico y acepte los términos del servicio. Después de hacerlo, se comunicará con el servidor Let's Encrypt y, a continuación, ejecutará un desafío para verificar que controla el dominio para el que está solicitando un certificado.certbotcertbot

La configuración se actualizará y Nginx se volverá a cargar para recoger la nueva configuración. terminará con un mensaje que le indicará que el proceso se realizó correctamente y dónde se almacenan sus certificados:

```
Output
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/TU_DOMINIO/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/TU_DOMINIO/privkey.pem
   Your cert will expire on 2022-08-08. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Los certificados se descargan, instalan y cargan. Intente volver a cargar su sitio web usando y observe el indicador de seguridad de su navegador. Debe indicar que el sitio está debidamente asegurado, generalmente con un icono de candado. Si prueba su servidor utilizando SSL Labs Server Test, obtendrá una calificación A.https://

Los certificados de Let's Encrypt solo son válidos durante noventa días. Esto es para alentar a los usuarios a automatizar su proceso de renovación de certificados. El paquete que instalamos se encarga de esto por nosotros agregando un temporizador systemd que se ejecutará dos veces al día y renovará automáticamente cualquier certificado que esté dentro de los treinta días posteriores al vencimiento.certbot

Puede consultar el estado del temporizador con:

    systemctl status certbot.timer

```
Output
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Mon 2022-08-08 19:05:35 UTC; 11s ago
    Trigger: Tue 2022-08-09 07:22:51 UTC; 12h left
   Triggers: ● certbot.service
```

Para probar el proceso de renovación, puede hacer una prueba en seco con:

    certbot renew --dry-run

Si no ve errores, ya está todo listo. Cuando sea necesario, Certbot renovará sus certificados y volverá a cargar Nginx para recoger los cambios. Si el proceso de renovación automática falla, Let's Encrypt enviará un mensaje al correo electrónico que especificó, advirtiéndole cuando su certificado esté a punto de caducar.

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

VS Code Server es un servicio que puede ejecutar en una máquina de desarrollo remoto, como su PC de escritorio o una máquina virtual (VM). Le permite conectarse de forma segura a esa máquina remota desde cualquier lugar a través de una URL vscode.dev, sin el requisito de SSH.

Para instalar VS Code Server, ejecute el siguiente lugar en la consola LXC.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/code-server.sh)"

VS Code Server Interfaz: IP:8680

### Administración del sistema con Webmin

Si prefiere administrar todos los aspectos de su Proxmox LXC desde una interfaz gráfica en lugar de la interfaz de la línea de comandos, Webmin podría ser adecuado para usted. Los beneficios incluyen actualizaciones automáticas diarias de seguridad, copias de seguridad y restauración, gestor de archivos con editor, panel de control web y monitoreo de sistemas preconfigurados con alertas de correo electrónico opcionales.

Para instalar Webmin System Administration (Screenshot), ejecute el siguiente en la consola LXC.

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/misc/webmin.sh)"

Webmin Interface: (IP:10000)
Nombre de usuario: root
Contraseña: root

Para desinstalar Webmin ejecute en la consola del contenedor `bash /etc/webmin/uninstall.sh`

### Sincronización de archivos

[Syncthing](https://syncthing.net/) es un programa de sincronización de archivos continuo. Sincroniza archivos entre dos o más ordenadores.

Para crear un nuevo LXC de sincronización de Proxmox, ejecute el siguiente comando en un shell de Proxmox.

bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/syncthing-v4.sh)"

Configuración predeterminada: 2GB de RAM - 8GB Almacenamiento - 2vCPU

Para el inicializar, ejecute `syncthing --gui-address=0.0.0.0:8384` en la consola del LXC, luego vaya a su_lxc_ip:8384. En los ajustes configuramos la dirección IP LXC bajo la GUI (también configura la GUI Authentication User y GUI Authentication Password) y lapestaña Connections. Luego guarde y reinicie el LXC.

Interfaz de sincronización: su_ip:8384

Actualizar Syncthing

apt update && apt upgrade -y

### Consola Unifi
#### UniFi Network Application LXC

La consola de red UniFi es un software que ayuda a administrar y monitorear las redes UniFi (Wi-Fi, Ethernet, etc.) al proporcionar una interfaz de usuario intuitiva y funciones avanzadas. Permite a los administradores de red configurar, monitorear y actualizar dispositivos de red, así como ver estadísticas de red, dispositivos de clientes y eventos históricos. El objetivo de la aplicación es hacer que la gestión de las redes UniFi sea más fácil y eficiente.

Ejecute el siguiente comando en una shell de Proxmox

    bash -c "$(wget -qLO - https://github.com/ctrbts/proxmox-scripts/raw/main/ct/unifi.sh)"

⚡ Default Settings: 2GB RAM - 8GB Storage - 2vCPU ⚡

UniFi Interface: (https)IP:8443
