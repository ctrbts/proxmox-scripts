# Proxmox Scripts

## Proxmox Tools

### Instalación posterior de Proxmox VE 7

El script dará opciones para deshabilitar Enterprise Repo, Agregar/Corregir fuentes PVE7, Habilitar Repo sin suscripción, Agregar Repo de prueba/Beta, Deshabilitar suscripción Nag, Actualizar Proxmox VE y Reiniciar PVE.

Ejecute lo siguiente en Proxmox Shell. ⚠️ SOLO PVE7

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"

Se recomienda responder y a todas las opciones.

### Escalado de CPU Proxmox

CPU Scaling Governor permite que el sistema operativo escale la frecuencia de la CPU hacia arriba o hacia abajo para ahorrar energía o mejorar el rendimiento. Gobernadores de escala genéricos

Ejecute lo siguiente en Proxmox Shell.

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/scaling-governor.sh)"

### Actualizador Proxmox LXC

Actualice todos los LXC de forma rápida y sencilla (Ubuntu, Debian, Devuan, Alpine Linux, CentOS-Rocky-Alma, Fedora, ArchLinux)

Ejecute lo siguiente en Proxmox Shell.

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/update-lxcs.sh)"

### Proxmox Discord Tema oscuro

Un tema oscuro para la interfaz de usuario web de Proxmox de Weilbyte

Ejecute lo siguiente en Proxmox Shell.

bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) instalar

Para desinstalar el tema, simplemente ejecute el script con el comando de desinstalación.

### Instalación posterior del servidor de copia de seguridad Proxmox

Instalación posterior del servidor de copia de seguridad Proxmox
El script dará opciones para deshabilitar Enterprise Repo, agregar/corregir fuentes de PBS, habilitar el repositorio sin suscripción, agregar repositorio de prueba, deshabilitar suscripción Nag, actualizar el servidor de copia de seguridad de Proxmox y reiniciar PBS.

Ejecute lo siguiente en Proxmox Shell. ⚠️ Servidor de respaldo Proxmox SOLAMENTE

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pbs-install.sh)"

Se recomienda responder y a todas las opciones.
