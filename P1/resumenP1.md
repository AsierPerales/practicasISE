# Práctica 1 : instalación de SO y configuración de almecenamiento

En esta práctica instalaremos y ejecutaremos en VirtualBox las imágenes ISO de dos SO Linux:

 - Debian 13
 - Alma 10
 
Ambas tendrán una configuración mínima (sin gráficos ni red por el momento) con tal de centrarnos en los requisitos personalizados de nuestros "clientes". Por un lado, Debian requerirá la configuración de un RAID1 gestionado con LVM (cifrado, por supuesto) de tres volúmenes lógicos (home, root y swap). En cambio, en Alma, además de configurar un RAID1 similar al de Debian nos pedirán otra instancia del servidor en que usaremos un disco exclusivamente para var, por lo que deberemos aprender a crear desde cero los niveles corresponientes con tal de extender el volumen de var.

De modo que las lecciones cubrirán lo siguiente:

- Lección 1 : Instalación y configuración del RAID1 cifrado de Debian
- Lección 2 : Instalación y configuración de VL con disco exclusivo (var) de Alma
- Lección 3 : Configuración de RAID1 con VL var cifrado en Alma


## Explicación L1

Como ya hemos dicho, debemos configurar (durante la instalación) un RAID1 cifrado.
Para ello, debemos:

- Añadir desde VirtualBox un "nuevo disco" del mismo tamaño que el original. Ambos serán de 10G y nos dirigiremos a ellos como sda o sdb (solid disk).
- Seguir los pasos requeridos para crear los VL boot (que no será necesario cifrar pues se encarga solemente del arranque), swap (1G por recomendación), home (500M) y / (resto de espacio) .

Los pasos son sencillos gracias al asistente de instalación, por lo que solo debemos particionar los discos físicos sda y sdb en dos (sin contar el GRUB, por supuesto): una para /boot del espacio ya mencionado, que irá en un LVM aparte y otra para los tres LV deseados. Una vez particionados fisicamente seberemos crear un volumen físico sobre la partición, sobre el que irá un grupo de volúmenes al cual podremos asignar los volúmenes lógicos necesarios.

Primero crearemos el LV /boot. Para ello, debemos crear un RAID1 por software que englobe las dos particiones respectivas de cada disco. Hecho esto, configuramos el tipo de sitema de ficheros que queremos y le decimos al instalador que lo monete en /boot. A este RAID1 lo llamaremos md0 (multi-device).

Después debemos crear el RAID1 interesante: md1. Bstará con seleccionar el espacio restante de cada disco para crear el RAID sobre este. 
Es aquí donde debemos tomar la primera "decisión" de diseño: ciframos el LVM (LUKS on LVM) o hacemos un LVM sobre el volumen ya cifrado? (LVM on LUKS); después de muchas noches sin dormir nos decantamos por la segunda opción, más cómoda y flexible para este escenario.

Cifrado el RAID md1, solo falta crear con el querido instalador el volumen físico que representa nuestro disco artificial RAID, para asignarle un grupo de volúmenes (llamado vgraid1) sobre el que podremos asignar el espacio necesario a cada volumen lógico, así como el sistema de ficheros y, por último pero no menos importante, el punto de montaje (swap, home y /).

Hecho esto el SO quedará instalado con la configuración deseada, y bastará el uso del comando lsblk para verificar la existencia de los dos discos físicos, con sus respectivos RAID1 por software (md0 y md1, al que aplicamos el encriptado) sobre los que creamos el LVM necesario para montar los volúmenes deseados.


## Explicación L2

Entendidos los conceptos de volúmenes físicos, loǵicos y grupos de volúmenes, haremos algo similar pero en el contexto de AlmaLinux.
Por desgracia, no nos encontramos con el instalador sencillo e intuitivo de Debian, por lo que deberemos crear el RAID1 poe software y usando comandos del sistema.
Recordemos que el cliente necesita un directorio /var con un disco exclusivo aparte por razones de espacio, por lo que tendremos que crear el LV exclusivo para luego extenderlo sobre la configurción que ya nos viene dada en la máquina.

Los pasos a seguir son idénticos a Debian, solo que debemos conocer los comandos necesarios para llevarlos a cabo:

- Crear las particiones físicas
- Crear el volumen físico
- Extender el grupo de volúmenes que viene instalado en Alma para incuir el nuevo
- Crear el volumen lógico destinado a /var y montarlo en este.

Es recomendable aprovechar para crear un usuario con privilegios de superusuario para prácicas posteriores.

Instalado el SO, usaremos fdisk para crear, sobre el disco de 4G previamente añadido en virtualbox la partición sdb1. Hecho esto ponemos en la terminal que nos sale n, w con la configuración por defecto, y salimos con q para verificar con lsblk que se creó.

Después es más sencillo aún: creamos sobre sdb1 el volumen físico con pvcreate, extendemos el grupo almalinux a sdb1 con vgextend creamos el volumen lógico (llamado new_var para evitar confusiones, pues ya hay un /var creado) con lvcreate, asignándole un tamaño de 3G e indicando el grupo en el que está (almalinux).

Ahora, lo complicado es montar new_var en /var, ya que debemos hacer todo lo que el instalador de Debian hacía por nosotros:

- Crear el sistema de archivos
- Montar new_var en /var
- Copiar (atómicamente!) los datos de /var en new_var para evitar pérdidas
- Utilizar fstab para indicar que ahora el directorio /var es nuestro directorio /new_var

Para crear el sistema de archivos, usaremos mkfs para asignarle una sistema ext4. Montaremos el volumen new_var en un directorio /new_var creado con mkdir.
Para la copia atómica de /var a /new_var, pondremos el sistema en modo mantenimiento con systemctl isolate rescue. Comprobamos que estamos "parados" con status, y hacemos un cp -a, y ya que estamos un ls -la para comprobar. 
En /etc/fstab deberemos añadir la ruta del volumen new_var junto con el directorio donde vamos a montarlo (/var) junto con el sistema de archivos que usaremos (ext4). Y si le damos a mount -a, montaremos todo lo que pone en fstab y podremos comprobar que new_var está montado en /var

Pero ojo! Hay que ser un profesional limpio, por lo que deberemos liberarnos del viejo /var. Tan sencillo como desmontar new_var de /var, "renombrar" /var a un directorio nuevo (/var_old) , volver a montar con la misma línea de fstab y ahora si que no tendremos más problemas.

## Explicación L3

En esta lección uniremos los dos casos anteriores para crear un RAID1 por software sobre un nuevo disco (ya que al estar en Alma, no podemos modificar la instalación) sobre el que crearemos nuestro propio grupo de volúmenes para tener un volumen /var que además estará cifrado, pero de forma inversa a como vimos en la lección 1 (es decir, en ver de cifrar el volumen físico y con él el grupo de volúmenes, ciframos solamente /var, que es el que nos interesa).


Los pasos son idénticos a los ya vistos, pero con un orden distinto:

- Añadir los dos discos
- Particiones físicas sdb1 y sdc1
- Crear el RAID1 (es muy fácil, basta con usar mdadm --crate, que es muy flexible)
- Crear volumen físico (pvcreate), CREAR grupo de volúmenes (vgextend) y crear volumen lógico (lvcreate)
- Encriptar el volumen new_var con cryptsetup (luksFormat y luego luksOpen)
- OJO! Hay que decirle a la tabla de cryptsetup (crypttab) el volumen que queremos montar cifrado, para que en el arranque ya sepa que está cifrado
- Montamos en /var ...
- Liberamos el viejo /var ...

Con esto podemos reiniciar el ordenador (asegurándonos de que 1: crypttab está escrito correcto 2: /new_var_crypt está bien montado 3: no se ha quedado el viejo /var por ahí) y esperar que todo haya salido bien.


