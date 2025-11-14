# Lección 3 P2 en Alma: Configuración de red, instalación de Apache, PHP y MariaDB

El objetivo de esta práctica es configurar correctamente ssh para acceso sin contraseña e instalar la pila LAMP.

Esto implica:

- Configurar la interfaz de red manualmente con `nmcli` y validar la conectividad
- Instalar y habilitar Apache
- Configurar claves SSH para acceder sin contraseña
- Instalar PHP y MariaDB, configurarlos y verificar su funcionamiento
- Crear un fichero index.php básico para pruebas



```
nmtui
```

Por si acaso se nos desactiva la interfaz de res (a veces pasa)



## SSH sin contraseña

Generamos las claves SSH:

```
ssh-keygen
```


Copiamos la clave pública al servidor:

```
ssh-copy-id -i .ssh/id_rsa.pub -o Port=22022 asier@192.168.56.105
```

Probamos conexión sin contraseña:

```
ssh asier@192.168.56.105 -p 22022
```

Conviene desactivar el acceso con contraseña para más seguridad, cosa que no he hecho para comprobar si fail2ban funcionaba :)


## Instalación y configuración de Apache (httpd)

Instalamos Apache:

```
sudo dnf install httpd
```

Comprobamos estado:

```
sudo systemctl status httpd
```

Lo habilitamos y arrancamos:

```
sudo systemctl enable httpd
sudo systemctl start httpd
```

Verificamos que responde:

```
curl 127.0.0.1
```

### Abrir puerto 80 en el firewall

En AlmaLinux (y Red Hat en general), **firewalld** bloquea los servicios hasta que los añadimos:

```
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```


## Instalación de PHP

Entramos al intérprete interactivo para comprobar si está instalado:

```
php -a
```

No lo está, así que lo instalamos:

```
sudo dnf install php
```

Volvemos a probar:

```
php -a
```


## Instalación de MariaDB y drivers

Instalamos MariaDB y su servidor:

```
sudo dnf install mariadb mariadb-server
```

Instalamos el módulo necesario para que PHP hable con MariaDB:

```
sudo dnf install php-mysqlnd
```

Intentamos acceder:

```
mysql -u root 
mysql -u root -p
```

Esto está muy mal en un servidor, así que jecutamos el script de seguridad:

```
mysql_secure_installation
```

Habilitamos y arrancamos el servicio:

```
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb
```

Probamos de nuevo:

```
mysql -u root -p
```


## Configuración de Apache + PHP

Editamos el fichero de configuración principal para modificar el **DirectoryIndex**:

```
sudo vi /etc/httpd/conf/httpd.conf
```

Buscamos la línea:

```
cat /etc/httpd/conf/httpd.conf | grep DirectoryIndex
```

Tras editar, reiniciamos Apache:

```
sudo systemctl restart httpd
sudo systemctl status httpd
```

---

## Creación del index.php

Creamos el fichero:

```
sudo touch /var/www/html/index.php
```

Lo editamos (con nano o vi, a mi me gusta más vi):

```
sudo vi /var/www/html/index.php
```

Comprobamos su contenido:

```
cat /var/www/html/index.php
```


