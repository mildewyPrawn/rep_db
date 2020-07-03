
# Table of Contents

1.  [Replicación de bases de datos](#orgf505f87)
    1.  [Cosas por hacer](#org054cdbf)
2.  [Reporte](#orgbc7dc4c)
        1.  [Integrantes:](#org37d6157)
    1.  [Introducción](#org95bfcf3)
    2.  [Instalación](#org5e617cb)
        1.  [Servidores:](#org41267df)
        2.  [Paquetes necesarios](#org10f7af6)
        3.  [Servidor Maestro](#orga7f66b8)
        4.  [Servidor Esclavo](#orga71b3b8)
        5.  [Virtual Hosts](#orgac031a0)
        6.  [MediaWiki](#org3f8c63c)
        7.  [phpMyAdmin](#org894eb95)
        8.  [Verificación](#org340bc45)
    3.  [Configuracion](#org8000a04)
        1.  [Instalación](#orgcf2284d)
        2.  [Configuración de 'Master'](#orgdc70e5a)
        3.  [Configuración de 'Slave'](#orgbf151eb)
        4.  [Configuración de 'MediaWiki'](#orge0dfcef)
        5.  [Configuración de 'phpMyAdmin'](#org592b8d3)
    4.  [Ligas de acceso](#orgf914c48)
        1.  [Servidor Maestro](#org6ffae47)
        2.  [Servidor Esclavo](#org943d8d5)
    5.  [Certificado SSL](#orge334c69)
    6.  [Configuración de Hosts Virtuales](#org8c29f49)
    7.  [Videos](#org14679d8)
    8.  [Errores frecuentes.](#org177cfc4)
    9.  [Bibliografía](#org9a530d4)


<a id="orgf505f87"></a>

# Replicación de bases de datos


<a id="org054cdbf"></a>

## Cosas por hacer

1.  [X] Instalar MySQL/MariaDB y PostgreSQL en un equipo que será el servidor
    maestro.
    1.  [X] MySQL
    2.  [X] PostgreSQL
2.  [X] Instalar MySQL/MariaDB y PostgreSQL en otro equipo que será el servidor
    esclavo.
    1.  [X] MySQL
    2.  [X] PostgreSQL
3.  [X] Instalar phpMyAdmin y PGadmin en el servidor maestro para monitorear la
    replicación.
4.  [X] Instalar dos instancias locales de MediaWiki y configurar una para que
    use MySQL/MariaDB y otra para que use PostgreSQL
    1.  [X] MySQL
    2.  [X] PostgreSQL
5.  [X] Verificar el servicio de base de datos maestro y verificar que la
    replicación lleve los datos al servidor esclavo.

El funcionamiento de estos servicios se puede ver en el siguiente diagrama:


<a id="orgbc7dc4c"></a>

# Reporte


<a id="org37d6157"></a>

### Integrantes:

-   Galeana Araujo Emiliano
-   Gladin García Angel Iván
-   Vazquez Rizo Paola


<a id="org95bfcf3"></a>

## Introducción

El trabajo se realizó en una máquina física y local, para posteriormente pasar
todo el trabajo al servidor de Amazon.

Recordemos que una replicación de base de datos sirve para copiar de forma
exacta en otra ubicación una instancia de la base de datos. Podemos usarla en
entornos distribuidos donde una sola base de datos tiene que ser utilizada y
actualizada en varios lugares.

Algunos beneficios en la replicación es que mejora la seguridad de los datos,
mejora el rendimiento (pues los múltiples accesos no saturan a los servidores),
y hay una mejora en la fiabilidad, pues podemos acceder a los datos si alguna
máquina tuviese errores.

Tipos de realización de bases de datos
Hay 3 principales tipos de replicación de bases de datos

-   Replicación de mezcla: los datos de dos o más bases de datos se combinan en
    una sola base de datos. En primer lugar se envía una copia completa de la
    base de datos. Luego el Sistema de Gestión de Base de Datos va comprobando
    los cambios que van apareciendo en los distintos nodos y a una hora
    programada o a petición los datos se sincronizan. Es sobre todo útil cuando
    cada nodo suele utilizar solo los datos que se actualizan allí pero que por
     circunstancias necesita tener también los datos de los otros sitios.

-   Replicación Instantánea: los datos de un servidor son simplemente copiados a
    otro servidor o a otra base de datos dentro del mismo servidor. Al copiarse
    todo no necesitas un control de cambios. Se suele utilizar cuando los datos
    cambian con  muy poca frecuencia.

-   Replicación Transaccional: primero se envía una copia completa de la base de
    datos y luego se van enviando de forma periódica (o a veces continua) las
    actualizaciones de los datos que cambian. Se utiliza cuando necesitas que
    todos los nodos con todas las instancias de la base de datos tengan los
    mismos datos a los pocos segundos de realizarse un cambio.

Los siguiente es necesario para el correcto funcionamiento del proyecto.

1.  PHP 7.3.14-1
2.  MySQL 8.0.20
3.  apache2 2.4.29
4.  phpMyAdmin


<a id="org5e617cb"></a>

## Instalación

Después de unos problemas técnicos con nuestra instancia de AWS, y conflictos
en la comunicación del equipo. Replicamos el proyecto en un par de  servidores
que compramos. Dichos servidores están alojados en [Digital Ocean](https://www.digitalocean.com/) y corren 
sobre `Ubuntu 16.04.6 (LTS) x64`. Necesitamos dos servidores, pues uno hará la
función de 'Master' y el otro la función de 'Slave'.

Las direcciones de dichos servidores son las siguientes:

-   Master: [104.248.53.119]
-   Slave:  [104.248.120.69]


<a id="org41267df"></a>

### Servidores:

La conexión a los servidores se hace mediante `ssh` de la siguiente manera:

    ssh root@<SERVIDOR>

Donde `<SERVIDOR>` es alguna dirección de las mencionadas anteriormente.

La clave para acceder a dichos servidores no la pondremos en el reporte, por
cuestiones de seguridad, sin embargo, en caso de ser requerida se pueden 
poner en contacto con algún integrante del equipo para brindarla.


<a id="org10f7af6"></a>

### Paquetes necesarios

En esta sección listaremos los paquetes y su manera de instalación.

Antes de iniciar cualquier cosa, vamos a actualizar.

    apt update && apt dist-upgrade

Lo primero en instalar será MySQL, es importante hacerlo tanto en el servidor
esclavo, como en el maestro

    apt-get install -y mysql-server

La bandera `-y` es solamente para responder `yes` a cualquier pregunta que se
nos haga a la hora de la instalación.

Por cuestiones de tiempo y optimización, optamos por utilizar `Tasakel`, 
pues ya viene con todo listo para una rápida instalación. De esa manera nos
evitamos instalar *Apache, MySQL, PHP* por separado. 

Utilizaremos `Taskel` para instalar `LAMP Stak` con los siguientes comandos.

    apt install tasksel
    tasksel install lamp-server

Necesitamos `mediaWiki`, el cuál vamos a instalar de la siguiente manera

    cd /var/www/html/foo
    sudo curl -O https://releases.wikimedia.org/mediawiki/1.33/mediawiki-1.33.0.tar.gz

Para usar phpMyAdmin, necesitamos descargarlo, lo podemos hacer con lo 
siguiente

    apt-get install phpmyadmin php-mbstring php-gettext


<a id="orga7f66b8"></a>

### Servidor Maestro

Necesitamos editar el siguiente archivo. Los cambios se pueden encontrar en 
la sección de *Configuración*.

     vi /etc/mysql/mysql.conf.d/mysqld.cnf
    
     Una vez editado, procedemos a reiniciar MySQL
    
     service mtysql restart
    
     Ahora procedemos a acceder a MySQL
    
     mysql -uroot
    
     Los comandos ingresados en la consola de MySQL, se pueden ver en la sección
     de /Configuración/.
    
     Hacemos un sanpshot usando  =mysqldump= con el siguiente comando
    
    mysqldump -uroot --all-databases --master-data > masterdump.sql

Y se lo mandamos al esclavo

    scp masterdump.sql 104.248.120.69:


<a id="orga71b3b8"></a>

### Servidor Esclavo

Necesitamos editar el siguiente archivo. Los cambios se pueden encontrar en 
la sección de *Configuración*.

    vi /etc/mysql/mysql.conf.d/mysqld.cnf
    
    Una vez editado, procedemos a reiniciar MySQL
    
    service mtysql restart
    
    Ahora procedemos a acceder a MySQL
    
    mysql -uroot
    
    Los comandos ingresados en la consola de MySQL, se pueden ver en la sección
    de /Configuración/.
    
    Ahora procedemos a restaurar la base de datos del maestro. Y entramos de 
    nuevo a la consola (Ver /Configuración/).
    
    mysql -uroot < masterdump.sql
    mysql -uroot


<a id="orgac031a0"></a>

### Virtual Hosts

Vamos a configurar nuestro host de distintas maneras, por lo que lo 
reemplazamos con

    cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/foo.conf

Creamos subdirectorios con

    mkdir -p /var/www/html/foo/{public_html, logs}

Les damos permisos

    sudo chmod -R 755 /var/www/html/foo/public_html

Deshabilitamos el host anterior y actualizamos `apache` con lo siguiente

    sudo a2dissite 000-default.conf
    systemctl reload apache2


<a id="org3f8c63c"></a>

### MediaWiki

Ver la sección *Paquetes necesarios* en este mismo apartado para ver el 
proceso de instalación.

Una vez instalado, procedemos a descomprimir el paquete.

`sudo tar -xvf mediawiki-1.33.0.tar.gz`

Ahora, moveremos el archivo descomprimido `mediawiki-1.33.0` al directorio 
`mediawiki`.

`sudo mv mediawiki-1.33.0/ public<sub>html</sub>/mediawiki/`

Como MediaWiki tiene que comunicarse con una base de datos para guardar la
información, vamos a crear una. (Ver *Configuración de MediaWiki*).


<a id="org894eb95"></a>

### phpMyAdmin

Ver la sección *Paquetes necesarios* en este mismo apartado para ver el 
proceso de instalación.

El proceso de instalación, carga la configuración en `/etc/apache2/conf-enabled/`
Necesitamos habilitar `mbstring` de la siguiente manera. 

    sudo phpenmod mbstring

Procedemos a reiniciar apache para guardar los cambios.

    sudo systemctl restart apache2

Ahora queremos configurar una contraseña para un usuario dedicado en `MySQL`
Las configuraciones se pueden ver en *Configuración phpMyAdmin*. Es 
importante recalcar que esto debe hacerse tanto en 'Master' como en 'Slave'.

    sydo -u mysql-p


<a id="org340bc45"></a>

### Verificación

Podemos ver que cada que escribimos en el maestro, se escribe automáticamente
en el esclavo, esto nos habla de una replicación instantánea.


<a id="org8000a04"></a>

## Configuracion


<a id="orgcf2284d"></a>

### Instalación

En general para cualquier cosa que descarguemos sirve el comando

`sudo apt-get install <paquete>`

`mediaWiki` se va a instalar con el siguiente comando.
`sudo curl -O <https://releases.wikimedia.org/mediawiki/1.33/mediawiki-1.33.0.tar.gz>`


<a id="orgdc70e5a"></a>

### Configuración de 'Master'

En el archivo `/etc/mysql/mysql.conf.d/mysqld.cnf` vamos a modificar el 
`bind-address`, por lo que basta con encontrar dicha línea y escribir lo 
siguiente.

`bind-address          = 104.248.53.119`

En el mismo archivo, también tenemos que descomentar las líneas de
`server-id` y `log-bin`.

1.  Consola de MySQL

    Lo primero es crear un usuario:
    
        create user 'repl'@'%' identified by 'slavepassword';
        
        El siguiente comando es para crear la replicación
        
        grant replication slave on *.* to 'repl'@'%';
        
        Para probar lo hecho anteriormente
        Creamos la base de datos.
        
        create database pets;
        create database pets.cats (name varchar(20));
        insert into pets.cats values ('fluffy');
        select * from pets.cats;
        exit


<a id="orgbf151eb"></a>

### Configuración de 'Slave'

En el archivo `/etc/mysql/mysql.conf.d/mysqld.cnf` vamos a modificar el 
`bind-address`, por lo que basta con encontrar dicha línea y escribir lo 
siguiente.

`bind-address          = 104.248.53.119`

En el mismo archivo, también tenemos que descomentar las líneas de
`server-id` y `log-bin`. El `server-id` tiene que quedar de la siguiente
manera.

`server-id             = 2`

1.  Consola de MySQL

    Lo siguiente es para ligar al esclavo con el maestro.
    
        CHANGE MASTER TO
        MASTER_HOST='104.248.53.119',
        MASTER_USER='repl',
        MASTER_PASSWORD='slavepassword';
        exit
    
    Con los siguientes comandos vamos a restaurar la base de datos del maestro.
    
        start slave;
    
    Podemos ver el estado con el siguiente comando
    
        show slave status\G;
    
    Y verificar que en efecto, la replicación está lista.


<a id="orge0dfcef"></a>

### Configuración de 'MediaWiki'

Antes de configurar MediaWiki como tal, vamos a crear una base de datos.

`sudo mysql -u root -p`

Ahora creamos una base de datos y un usuario. la base de datos llevará por 
nombre `my_wiki`, el usuario será `media_wiki` y la contraseña no la 
pondremos, pero en caso de requerirla, de nuevo, se puede contactar a 
cualquier miembro del equipo.

`CREATE DATABASE my<sub>wiki</sub>;
CREATE USER 'media<sub>wiki</sub>'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON my<sub>wiki</sub>.\* TO 'media<sub>wiki</sub>'@'localhost' IDENTIFIED BY 'password';`

Una vez hecho lo anterior, ahora si, procedemos a configurar `mediaWiki`.

Vamos a ir a la url `foo/mediawiki/` y ahí encontraremos un link con la 
leyenda: `Please set up the wiki first`, vamos a seleccionar el nombre de la 
base y la contraseña que definimos anteriormente. Una vez terminada la 
configuración `mediaWiki` va a crear un archivo (`LocalSettings.php`), el 
cual contiene las configuraciones de la instalación. 

Como último paso, vamos a mover el arhivo antes mencionado y le vamos a 
restringir el acceso de la siguiente manera.

`mv LocalSettings.php /var/www/html/foo/public<sub>html</sub>/mediawiki
sudo chmod 700 /var/www/html/example.com/public<sub>html</sub>/media/wiki/LocalSettings.php`

La contraseña de `mediaWiki` no la pondremos en este reporte, pero de nuevo,
de ser requerida, cualquier miembro del equipo puede brindarla.


<a id="org592b8d3"></a>

### Configuración de 'phpMyAdmin'

Podermos crear un usuario y una contrseña nuevos, pero vamos a usar los 
mismos que para la base de datos de `MySQL` vamos a condecer los privilegios
necesarios y por último, nos salimos del shell de `MySQL`. La cuál se 
encuentra arriba.

Ahora podemos accesar a la interfaz de web visitando `http://<dominio>/phpmyadmin`


<a id="orgf914c48"></a>

## Ligas de acceso

Por el momento, por falta de comunicación, solo tenemos una instancia de
`mediaWiki` con una base de datos, por lo que nos haría falta crearla con
`postgres`.


<a id="org6ffae47"></a>

### Servidor Maestro

-   Podemos ver una instancia de `mediaWiki` en [MediaWiki - MySQL](http://104.248.53.119/mediawiki/)
-   Podemos ver la instancia de `phpMyAdmin` [phpMyAdmin](http://104.248.53.119/phpmyadmin/)


<a id="org943d8d5"></a>

### Servidor Esclavo

-   Podemos ver la instancia de `phpMyAdmin` en [phpMyAdmin](http://104.248.120.69/phpmyadmin/)


<a id="orge334c69"></a>

## Certificado SSL

Parece que necesitamos un nombre de dominio para poder realizar esta parte, 
sin embargo, suponiendo que lo tenemos, bastaría con realizar los siguientes
comandos.

    apt-get install certbot python-certbot-apache 
    sudo certbot --apache


<a id="org8c29f49"></a>

## Configuración de Hosts Virtuales

Antes de comenzar, necesitamos instalar apache:

    apt-get install apache2

Después de la instalación, lo primero que vamos a hacer es crear la estructura
de nuestros directorios, `public_html` será el directorio que contenga 
nuestros archivos, también vamos a cambiar de propietario, y actualizar 
permisos todo eso se puede ver en los siguientes comandos.

    mkdir -p /var/www/otro-db-master.redes.tonejito.cf/
    sudo chown -R $USER:$USER /var/www/otro-db-master.redes.tonejito.cf/
    sudo chmod -R 755 /var/www

Ahora vamos a crear el contenido de nuestra página, por lo que vamos a crear
el archivo `index.html`. El archivo es sencillo puede verse en las carpetas 
mencionadas anteriormente una vez que se acceda al servidor.

Para crear los hosts virtuales, vamos a copiar el siguiente archivo, y 
editarlo.

    cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/otro-db-master.redes.tonejito.cf.conf
    vim /etc/apache2/sites-available/otro-db-master.redes.tonejito.cf.conf

Las líneas editadas se pueden visualizar en ambos archivos, pero hay que 
cambiar *DocumentRoot* y agregar *ServerName, ServerAlias* y agregar nuestro
`DNS`.

Para habilitar nuestro host vitual podemos usar `a2ensite` de la siguiente 
forma. Que habilita nuestra nueva configuración, y deshabilita la anterior (la
que se tenía por default).

    sudo a2ensite otro-db-master.redes.tonejito.cf.conf
    sudo a2dissite 000-default.conf

Una vez hecho esto, reestablecemos el servicio de `Apache` de la siguiente 
manera.

    sudo systemctl restart apache2

Y listo, hay que replicarlo en el otro servidor (Master o Slave) según sea el 
caso.


<a id="org14679d8"></a>

## Videos

La lista de reproducción para nuestros videos, se puede encontrar en el 
siguiente [link](https://www.youtube.com/playlist?list=PLH4x3V-hHwbXwkcet0yYi-CYQWTQxjU9V).


<a id="org177cfc4"></a>

## Errores frecuentes.

El mayor error que tuvimos fue la falta de comunicación, en un inicio, no 
pensamos que el hecho de que solo una persona tuviera los archivos de 
configuración nos perjudicaría. Un error que tuvimos (Cuando lo hicimos con 
AWS). fue que no recordábamos el passphrase para conectarnos.

Ya trabajando en `Digital Ocean` lo más complicado fue vimos fue intentar 
realizarlo con `postgres`, pero en la entrega del video ya estará. Otro error
Otro error que tuvimos fue que al momento de querer realizar una configuración
, la documentación de `MySQL` dice que hay que editar el siguiente archivo
`Domain Forwarding`, pero tuvimos problemas con eso y buscando, encontramos en
un foro que para versiones recientes, había que modificar el siguiente archivo
`/etc/mysql/mysql.conf.d/mysqld.cnf`, fue de las cosas más problemáticas que
tuvimos.   


<a id="org9a530d4"></a>

## Bibliografía

-   [problema con phpMyAdmin](https://askubuntu.com/questions/387062/how-to-solve-the-phpmyadmin-not-found-issue-after-upgrading-php-and-apache)
-   [Instalación](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04)
-   [instalación de phpMyAdmin](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-16-04)
-   [LAMP Stack](https://www.linode.com/docs/web-servers/lamp/how-to-install-a-lamp-stack-on-ubuntu-18-04/)
-   [mediaWiki Ubuntu](https://www.linode.com/docs//websites/wikis/install-mediawiki-on-ubuntu-1804/)
-   [Configuración MySQL](https://www.youtube.com/watch?v=JXDuVypcHNA)

