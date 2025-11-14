# Lección 1 P2 en Alma : Instalación y configuración de servicios

El objetivo es instalar (habiendo hecho la configuración de red pertinente) y configurar el servicio SSH para acceder de forma segura al servidor remotamente.

Esto implica:

- Efectuar la configuración AUTOMÁTICA de red en el servidor
- Instalar con el gestor de paquetes SSH
- Configurar el servidor y el cliente SSH
- Deshabilitar el login de root
- Cambiar los puertos por defecto

## Configuración automática de red 

Normalmente utilizamos el NetworkManager y sus herramientas derivadas nmcli y nmtui. En mi caso utilicé nmtui para activar las dos interfaces de red ya definidas en virtualbox (host-only para comunicación entre servidores y ethernet para paquetes y lo demás).

También está bien probar a hacer ping tanto a la máquina Debian (192.168.56.105) como al adaptador (192.168.56.1).

Con esto podemos instalar SSH y hacer que las máquinas se conozcan.

## Instalar y configurar SSH

Por suerte, Alma ya dispone del servicio ssh por defecto, que podemos comprobar con systemctl status sshd

Para probar la conexión : ssh asier@192.168.56.105


## Desactivar el acceso de root

Basta con editar el ficher /etc/ssh/sshd_config

En él descomentamos PermitRootLogin y le ponemos que no. También podemos deshabilitar el acceso por contraseña, activar PAM (Pluggable Authentication Modules).

Conviene comprobar con : ssh root@192.168.56.105 -v (para ver porqué no nos deja)

## Cambiar puertos por defecto

Todo el mundo sabe que el puerto por defecto de ssh es el 22, por lo que conviene poner otro puerto de escucha que menos gente conozca para evitar visitas indeseadas.

En Alma y todos los sistema que usan SELinux (oséase, todos los de Red Hat) tenemos que usar semanage cada vez que queremos cambiar el puerto por defecto en sshd_config. Irónicamente, no viene instalado, por lo que tendremos que manejar el gestor de paquetes de la forma:

- dnf provides semanage
- encontramos el paquete que necesita semanage (policycoreutils-python-utils)
- instalamos ambos 

Habilitamos con semanage port -a -t ssh_port_t -p tcp 22022 (o el puerto que queramos y esté libre)

Y no nos olvidamos de dejar constancia de ello en sshd_config.


Con esto tenemos acceso por medio de shh seguro!


## El firewall

La última línea era mentira, ya que el firewall nativo de Alma, como es típico de Red Hat, no nos deja pasar.
Por suerte bastará con añadir el puerto con firewall-cmd --add-port 22022/tcp.

Y ahora sí tenemos ssh seguro!




