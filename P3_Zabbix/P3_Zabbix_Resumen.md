# P3-MonitProf

```
Realice una instalación de Zabbix 7.4 en su servidor con Debian 13 y configure 
para que se monitorice a él mismo y para que monitorice a la máquina con 
Alma Linux.
Puede configurar varios parámetros para monitorizar, uso de CPU, memoria, etc. 
pero debe configurar de manera obligatoria, como mínimo, la monitorización de 
los servicios SSH y HTTP.
```

https://www.zabbix.com/documentation/current/en/manual

# Instalando Zabbix

*En el apartado de instalación del manual nos proveen la dirección del repositorio donde se encuentra el código fuente de Zabbix:*

```bash
sudo apt install git
git clone https://git.zabbix.com/scm/zbx/zabbix.git
```

*Pero eso (por experiencia personal) es muy engorroso, así que vamos al apartado 4 : Instalación por paquetes, donde nos lleva a la página de instalación oficial:*

[https://www.zabbix.com/download](https://www.zabbix.com/download)

![imagen.png](P3-MonitProf/imagen.png)

*Así es mucho mejor, pues abajo nos aparecen los comandos necesarios:*

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/ \
 zabbix-release/zabbix-release_latest_7.4+debian13_all.deb

dpkg -i zabbix-release_latest_7.4+debian13_all.deb

apt update 

```

*Y ya tenemos zabbix, solo faltan las dependencias:*

- Servidor (con MySQL)
- Frontend (con PHP y Apache)
- Scripts de configuración
- Agente de Zabbix

Habiendo hecho dpkg y update antes !!!! 

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf 
						zabbix-sql-scripts zabbix-agent
```

# Configurando zabbix

Zabbix normalmente trabaja sobre una base de datos que tenemos que configurar nosotros.

Por eso necesitamos MySQL, concretamente MariaDB para Debian:

```bash
apt install mariadb-server mariadb-client
mariadb-secure-installation

mysql -u root -p
```

*Y creamos la base de datos y el usuario zabbix con sus privilegios tal como nos especifica:*

```bash
## Dentro de mysql ...
create database zabbix character set utf8mb4 collate utf8mb4_bin;

create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;

set global log_bin_trust_function_creators = 1;
```

Importamos la base de datos inectando los comando a MySQL SIN INTERRUMPIR (tendríamos que crear otra vez la base de datos, no es que me haya pasado ni mucho menos):

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql 
--default-character-set=utf8mb4 -uzabbix -p zabbix 
```

Creo que esto es para que no midifiquemos la base de datos (ya que estamos comprobamos):

```
mysql> set global log_bin_trust_function_creators = 0;
mysql> use zabbix;
mysql> show tables;
mysql> quit;
```

Vamos a /etc/zabbix/zabbix_server.conf :

```bash
## Es la que pusimos al configurar la DB
DBPassword=password
```

NO NOS OVLIDAMOS DEL FIREWALL!!!

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Vamos a terminar la instalación:

```bash
http://192.168.56.105/zabbix

## En login:
Usuario: Admin
Contraseña: practicas,ise (la cambié, es zabbix por defecto)
```

# Usando Zabbix para Alma

Añadir un host es muy sencillo:

[https://www.zabbix.com/documentation/7.4/en/manual/quickstart/host](https://www.zabbix.com/documentation/7.4/en/manual/quickstart/host)

Lo difícil es encontrar el repo necesario para instalar el agente en el host objetivo:

[https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/](https://repo.zabbix.com/zabbix/6.4/rhel/8/x86_64/)

Ahora es tan sencillo como:

1. **Configurar el Agente** (`/etc/zabbix/zabbix_agentd.conf`):
    
    ```
    Server=192.168.56.105
    ServerActive=192.168.56.105
    Hostname=AlmaLinux
    
    ```
    
2. **Iniciar y Habilitar el Agente**:
    
    ```bash
    sudo systemctl restart zabbix-agent
    sudo systemctl enable zabbix-agent
    
    ```
    
3. **Configurar el Firewall** (si está activo) para permitir el puerto **10050 y 10051**:
    
    ```bash
    sudo firewall-cmd --permanent --add-port=10050/tcp  
    sudo firewall-cmd --permanent --add-port=10051/tcp  
    sudo firewall-cmd --reload
    
    ```
    
4. **Añadir el Host en Zabbix Server** a través de la interfazweb:
    - **Host name**: Debe coincidir con el `Hostname` configurado en el agente.
    - **IP address**: La dirección IP de la máquina donde está el agente.
    - **Template**: Asignar un template adecuado, como **Template OS Linux by Zabbix agent**.

# Añadiendo datos de SSH y HTTP

Para añadir plantillas predefinidas en Zabbix para monitorizar más cosas vamos a 

[https://www.zabbix.com/integrations](https://www.zabbix.com/integrations)

→ Equipos → (En apartado plantillas)

- `Template OS Linux by Zabbix Agent`
- `Template App Apache by Zabbix Agent`

Aún asi no encontraba la de SSH, por lo que me metí en la página de repos de Zabbix, literalmente copié el archivo (cambié a mano la versión y quité la fecha), lo importé desde el menú de plantillas y ya me aparecía en las plantillas disponibles:

[https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/app/ssh_service/template_app_ssh_service.yaml?at=release%2F6.2](https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/app/ssh_service/template_app_ssh_service.yaml?at=release%2F6.2)

Y ya monitorizaba todo!
