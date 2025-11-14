# Lección 1 P2 en Debian : Instalación y configuración de servicios

El objetivo es instalar ssh (teniendo la condiguración de red a punto) y hacer ciertas modificaciones para que sea más robusto, tales como:

- Deshabilitar el login de root
- Cambiar los puertos por defecto
- Gestionar el firewall

## Configuración de red en Debian

En Debian utilizamos NetPlan para la configuración automática de red, que se basa en ficheros YAML donde especificamos las interfaces de red (enp0s3 con dhcp para conexiones fuera de la red local y enp0s8 SIN DHCP para tener una ip fija y que Alma y el host puedan comunicarse).

Basta pues con editar /etc/netplan/00-installer-config.yaml

## Instalación ssh

Ahora que tenemos red, podemos instalar el paquete openssh-server, el cual por desgracia no viene integrado como en Alma.

Comprobamos con systemctl status sshd, ya que normalmente nos lo dan desactivado, y hay que hacer enable y start para que empiece a escuchar.

## Deshabilitar acceso de root y cambiar puertos por defecto

Basta con editar /etc/ssh/sshd_config:

- PermitRootLogin no
- Port 22022

Recomiendo personalmente reiniciar el servicio.

## El firewall

Al contrario que en Alma, el firewall de Debian (ufw) es más bien laxo, por lo que conviene hacer lo siguiente:

- ufw enable
- ufw allow 22022


