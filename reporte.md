
# Table of Contents

1.  [Replicación de bases de datos](#org8c1d4cb)
    1.  [Cosas por hacer](#org5e82177)
2.  [Reporte](#orgc7885e0)
        1.  [Integrantes:](#orgf147917)
    1.  [Introducción](#org4498bba)
    2.  [Instalación](#org00ebc7c)
        1.  [Servidores:](#orgf029183)
        2.  [Paquetes necesarios](#org948ad01)
        3.  [Servidor Maestro](#org86393c2)
        4.  [Servidor Esclavo](#org6a52d03)
        5.  [Virtual Hosts](#orga63ffe0)
        6.  [MediaWiki](#org5cb8c38)
        7.  [phpMyAdmin](#orgb4d6571)
        8.  [Verificación](#org20d51ea)
        9.  [PostgreSQL, phpPgAdmin](#org06b4473)
        10. [Apache Web Server](#org0043266)
    3.  [Configuracion](#org1c54dfb)
        1.  [Instalación](#orgd6ef968)
        2.  [Configuración de 'Master' (MySQL)](#org785c34c)
        3.  [Configuración de 'Slave'  (MySQL)](#org89f0cc4)
        4.  [Configuración de 'MediaWiki'](#orgd7b9c45)
        5.  [Configuración de 'phpMyAdmin'](#org70ada32)
        6.  [Configuración de 'postgreSQL'](#orgeb213bc)
    4.  [Ligas de acceso](#orga96cee1)
        1.  [Servidor Maestro](#org90f5e02)
        2.  [Servidor Esclavo](#org20bce52)
    5.  [Certificado SSL](#orgefa5f6d)
    6.  [Configuración de Hosts Virtuales](#org0bd0c44)
    7.  [Videos](#orgfa7c2c6)
    8.  [Errores frecuentes.](#org7e20172)
    9.  [Bibliografía](#org49a53a9)


<a id="org8c1d4cb"></a>

# Replicación de bases de datos


<a id="org5e82177"></a>

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

![Diagrama](./img/diagrama.png)

<a id="orgc7885e0"></a>

# Reporte


<a id="orgf147917"></a>

### Integrantes:

-   Galeana Araujo Emiliano
-   Gladin García Angel Iván
-   Vazquez Rizo Paola


<a id="org4498bba"></a>

## Introducción

En un inicio, el trabajo se realizó en una máquina física y local, para 
posteriormente pasar todo el trabajo al servidor de Amazon. Debido a fallas en
la comunicación, el trabajo se realizó en servidores de `digitalOcean`, en 
unos de prueba y posteriormente a los servidores que entregamos con este trabajo.

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


<a id="org00ebc7c"></a>

## Instalación

Después de unos problemas técnicos con nuestra instancia de AWS, y conflictos
en la comunicación del equipo. Replicamos el proyecto en un par de  servidores
que compramos. Dichos servidores están alojados en [Digital Ocean](https://www.digitalocean.com/) y corren 
sobre `Ubuntu 16.04.6 (LTS) x64`. Necesitamos dos servidores, pues uno hará la
función de 'Master' y el otro la función de 'Slave'.

Las direcciones de dichos servidores son las siguientes:

-   Master: [104.248.53.119]
-   Slave:  [104.248.120.69]


<a id="orgf029183"></a>

### Servidores:

La conexión a los servidores se hace mediante `ssh` de la siguiente manera:

    ssh root@<SERVIDOR>

Donde `<SERVIDOR>` es alguna dirección de las mencionadas anteriormente.

La clave para acceder a dichos servidores no la pondremos en el reporte, por
cuestiones de seguridad, sin embargo, en caso de ser requerida se pueden 
poner en contacto con algún integrante del equipo para brindarla.


<a id="org948ad01"></a>

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


<a id="org86393c2"></a>

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


<a id="org6a52d03"></a>

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


<a id="orga63ffe0"></a>

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


<a id="org5cb8c38"></a>

### MediaWiki

Ver la sección *Paquetes necesarios* en este mismo apartado para ver el 
proceso de instalación.

Una vez instalado, procedemos a descomprimir el paquete.

> sudo tar -xvf mediawiki-1.33.0.tar.gz

Ahora, moveremos el archivo descomprimido `mediawiki-1.33.0` al directorio 
`mediawiki`.

> sudo mv mediawiki-1.33.0/ public<sub>html</sub>/mediawiki/

Como MediaWiki tiene que comunicarse con una base de datos para guardar la
información, vamos a crear una. (Ver *Configuración de MediaWiki*).


<a id="orgb4d6571"></a>

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


<a id="org20d51ea"></a>

### Verificación

Podemos ver que cada que escribimos en el maestro, se escribe automáticamente
en el esclavo, esto nos habla de una replicación instantánea.


<a id="org06b4473"></a>

### PostgreSQL, phpPgAdmin

Lo que se quiere lograr con la replicación es que todo lo que se encuentra en
nuestra base de datos de el ****master**** se replique automáticamente al
****slave****, de forma que estén sincornizados. Para esto se debe configurar. 
Usaremos la  versión 10.

La replicación con `postgreSQL` (Al igual que la otra), busca que todo lo que
se encuentre en la base de datos en '**master**' se replique automáticamente en
'**slave**'. Para lograr esto, tenemos que configurar `postgreSQL`, de forma 
que usaremos la versión 10.

Tuvimos problemas con la versión en los servidores (Ver *Errores frecuentes*)
por lo que los siguientes pasos se siguieron para poder descargar la versión
correcta en ambos servidores.

Agregamos `ostgreSQL` a nuestro repositorio de `Ubuntu` con el siguiente
comando.

    echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main 11" |
    sudo tee /etc/apt/sources.list.d/pgsql.list

Después agregamos la llave GPG, y actualizamos el repositorio `apt`.

    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    sudo apt update

Después instalaremos con:

    sudo apt install postgresql-10

Una vez con los paquetes ver *Configuración-Configuración de 'PostgreSQL'*
para ver lo que se hizo.


<a id="org0043266"></a>

### Apache Web Server

En esta parte, vamos a realizar la configuración para `phpPgAdmin`, la cuál 
es generada automáticamente durante la instalación. Lo primero es crear el 
archivo `phopgadmin.conf`. Añadiendo la siguiente línea al inicio de nuestro
archivo `Alias /pgsqladminlogin /usr/share/phppgadmin`, comentamos la línea
`Requiere local` y agregamos `Allow From all`. Una vez hecho todo eso, 
podemos guardar, cerrar y reiniciar el servicio.

    systemctl restart apache2


<a id="org1c54dfb"></a>

## Configuracion


<a id="orgd6ef968"></a>

### Instalación

En general para cualquier cosa que descarguemos sirve el comando

> sudo apt-get install <paquete>

`mediaWiki` se va a instalar con el siguiente comando.

> sudo curl -O <https://releases.wikimedia.org/mediawiki/1.33/mediawiki-1.33.0.tar.gz>    


<a id="org785c34c"></a>

### Configuración de 'Master' (MySQL)

En el archivo `/etc/mysql/mysql.conf.d/mysqld.cnf` vamos a modificar el 
`bind-address`, por lo que basta con encontrar dicha línea y escribir lo 
siguiente.

> bind-address          = 104.248.53.119

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


<a id="org89f0cc4"></a>

### Configuración de 'Slave'  (MySQL)

En el archivo `/etc/mysql/mysql.conf.d/mysqld.cnf` vamos a modificar el 
`bind-address`, por lo que basta con encontrar dicha línea y escribir lo 
siguiente.

> bind-address          = 104.248.53.119

En el mismo archivo, también tenemos que descomentar las líneas de
`server-id` y `log-bin`. El `server-id` tiene que quedar de la siguiente
manera.

> server-id             = 2

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


<a id="orgd7b9c45"></a>

### Configuración de 'MediaWiki'

Antes de configurar MediaWiki como tal, vamos a crear una base de datos.

> sudo mysql -u root -p

Ahora creamos una base de datos y un usuario. la base de datos llevará por 
nombre `my_wiki`, el usuario será `media_wiki` y la contraseña no la 
pondremos, pero en caso de requerirla, de nuevo, se puede contactar a 
cualquier miembro del equipo.

> CREATE DATABASE my<sub>wiki</sub>;
> 
> CREATE USER 'media<sub>wiki</sub>'@'localhost' IDENTIFIED BY 'password';
> 
> GRANT ALL ON my<sub>wiki</sub>.\* TO 'media<sub>wiki</sub>'@'localhost' IDENTIFIED BY 'password';


Una vez hecho lo anterior, ahora si, procedemos a configurar `mediaWiki`.

Vamos a ir a la url `foo/mediawiki/` y ahí encontraremos un link con la 
leyenda: `Please set up the wiki first`, vamos a seleccionar el nombre de la 
base y la contraseña que definimos anteriormente. Una vez terminada la 
configuración `mediaWiki` va a crear un archivo (`LocalSettings.php`), el 
cual contiene las configuraciones de la instalación. 

Como último paso, vamos a mover el arhivo antes mencionado y le vamos a 
restringir el acceso de la siguiente manera.

> mv LocalSettings.php /var/www/html/foo/public<sub>html</sub>/mediawiki
> 
> sudo chmod 700 /var/www/html/example.com/public<sub>html</sub>/media/wiki/LocalSettings.php

La contraseña de `mediaWiki` no la pondremos en este reporte, pero de nuevo,
de ser requerida, cualquier miembro del equipo puede brindarla.


<a id="org70ada32"></a>

### Configuración de 'phpMyAdmin'

Podermos crear un usuario y una contrseña nuevos, pero vamos a usar los 
mismos que para la base de datos de `MySQL` vamos a condecer los privilegios
necesarios y por último, nos salimos del shell de `MySQL`. La cuál se 
encuentra arriba.

Ahora podemos accesar a la interfaz de web visitando `http://<dominio>/phpmyadmin`


<a id="orgeb213bc"></a>

### Configuración de 'postgreSQL'

Una vez instalado `postgreSQL`, necesitamos una contraseña para este, el 
siguiente comando nos ayuda con esto. La contraseña igual que las demás, no
se pondrá aquí, pero se puede contactar a cualquier persona del equipo en 
caso de requerirse.

    sudo passwd postgres

Es importante hacer lo mismo en el `servidor slave`.

1.  Master

    Ahora vamos a iniciar sesión en `postgreSQL` en el servidor `master`.
    
        su - postgres
    
    Una vez dentro, vamos a crear un usuario para la replicación:
    
        psql -c "CREATE USER replication REPLICATION LOGIN CONNECTION LIMIT 1 ENCRYPTED PASSWORD password;"
    
    Ahora podemos salir de `postgreSQL` y necesitamos configurar el siguiente 
    archivo `/etc/postgresql/10/main/pg_hba.conf`, en el cuál necesitamos agregar 
    la siguiente línea al final del archivo
    `host    replication     replication   DIRECCIÓN/24   md5`. Donde '**DIRECCIÓN**'
    es la dirección de `slave` (Ver *Instalación* para las direcciones).
    
    Ahora abrimos la configuración de `postgreSQL`, la cual se encuentra en el
    siguiente archivo: `/etc/postgresql/10/main/postgresql.conf`. Tenemos que 
    cambiar las siguiente líneas y descomentarlas (de ser necesario).
    
    -   `listen_addresses = 'localhost, DIR_MASTER'` con DIR<sub>MASTER</sub> la dirección del servidor `master`
    -   `wal_level = replica`
    -   `max_wal_senders = 10`
    -   `wal_keep_segments = 64`
    
    Una vez hecho esto, ponemos el siguiente comando para resetear el servidio de
    `postgreSQL`.
    
        systemctl restart postgresql

2.  Slave

    Una vez dentro del servidor `Slave`, vamos a ingresar sesión, y detener el 
    servicio en el servidor. Los siguientes comandos hacen exactamente eso.
    
        su - postgres
        systemctl stop postgresql
    
    Una vez hecho lo anterior, vamos a editar el mismo archivo que en `master`,
    esto es `/etc/postgresql/10/main/pg_hba.conf`, y vamos a agregar la siguiente
    línea, igual que hicimos en `master`. Donde '**DIRECCIÓN**' es la dirección de
    `master`.
    
        host    replication     replication     DIRECCIÓN/24   md5
    
    Ahora vamos a abrir la configuración, la cual se encuentra en 
    `/etc/postgresql/10/main/postgresql.conf` Tenemos que cambiar las siguiente 
    líneas y descomentarlas (de ser necesario).
    
    -   `listen_addresses = 'localhost, DIR_SLAVE'` con DIR<sub>SLAVE</sub> la dirección del servidor `slave`
    -   `wal_level = replica`
    -   `max_wal_senders = 10`
    -   `wal_keep_segments = 64`
    -   `hot_standby = on`
    
    Ahora vamos a nuestro `data_directory` y borramos todo lo que contenga.
    
        cd /var/lib/postgresql/10/main
        rm -rfv *
    
    Y podemos copiar los archivos de `master`. Recordemos que '**DIR<sub>MASTER</sub>**' es 
    la dirección de nuestro servidor `master`.
    
        pg_basebackup -h DIR_MASTER -D /var/lib/postgresql/10/main/ -P -U replication --wal-method=fetch
    
    Ahora creamos el archivo `recovery.conf`, y agregamos lo siguiente:
    
        standby_mode          = 'on'
        primary_conninfo      = 'host=DIR_MASTER192.168.199.137 port=5432 user=replication password=password'
        trigger_file = '/tmp/MasterNow'
    
    Una vez hecho esto, podemos iniciar el servicio de `postgreSQL`:
    
        systemctl start postgresql

3.  Tests

    Si todo salió bien, la replicación ya estaría, faltaría probarla, para esto
    en `master` basta con `CREATE TABLE...` y agregamos valores, luego vamos a 
    `slave` y verificamos que esos valores se encuentren reflejados
    `SELECT * FROM ...`.


<a id="orga96cee1"></a>

## Ligas de acceso

Por el momento, por falta de comunicación, solo tenemos una instancia de
`mediaWiki` con una base de datos, por lo que nos haría falta crearla con
`postgres`.


<a id="org90f5e02"></a>

### Servidor Maestro

-   Podemos ver una instancia de `mediaWiki` en [MediaWiki - MySQL](http://104.248.53.119/mediawiki/)
-   Podemos ver la instancia de `phpMyAdmin` [phpMyAdmin](http://104.248.53.119/phpmyadmin/)


<a id="org20bce52"></a>

### Servidor Esclavo

-   Podemos ver la instancia de `phpMyAdmin` en [phpMyAdmin](http://104.248.120.69/phpmyadmin/)


<a id="orgefa5f6d"></a>

## Certificado SSL

Parece que necesitamos un nombre de dominio para poder realizar esta parte, 
sin embargo, suponiendo que lo tenemos, bastaría con realizar los siguientes
comandos.

    apt-get install certbot python-certbot-apache 
    sudo certbot --apache


<a id="org0bd0c44"></a>

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


<a id="orgfa7c2c6"></a>

## Videos

La lista de reproducción para nuestros videos, se puede encontrar en el 
siguiente [link](https://www.youtube.com/playlist?list=PLH4x3V-hHwbXwkcet0yYi-CYQWTQxjU9V).


<a id="org7e20172"></a>

## Errores frecuentes.

El mayor error que tuvimos fue la falta de comunicación, en un inicio, no 
pensamos que el hecho de que solo una persona tuviera los archivos de 
configuración nos perjudicaría. Un error que tuvimos (Cuando lo hicimos con 
AWS). fue que no recordábamos el passphrase para conectarnos.

Ya trabajando en `Digital Ocean` lo más complicado fue vimos fue intentar 
realizarlo con `postgres`, pues tuvimos que realizarlo varias veces, incluso 
en servidores de prueba porque nos fallaban algunas cosas de la conexión. Otro
error que tuvimos fue que al momento de querer realizar una configuración, la
documentación de `MySQL` dice que hay que editar el siguiente archivo
`Domain Forwarding`, pero tuvimos problemas con eso y buscando, encontramos en
un foro que para versiones recientes, había que modificar el siguiente archivo
`/etc/mysql/mysql.conf.d/mysqld.cnf`, fue de las cosas más problemáticas que
tuvimos.

Lo último (Aunque no fue error como tal) fue tener dos instancias de 
`mediaWiki` juntas, pero después de checar algunos foros lo logramos.


<a id="org49a53a9"></a>

## Bibliografía

-   [problema con phpMyAdmin](https://askubuntu.com/questions/387062/how-to-solve-the-phpmyadmin-not-found-issue-after-upgrading-php-and-apache)
-   [Instalación](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04)
-   [instalación de phpMyAdmin](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-16-04)
-   [LAMP Stack](https://www.linode.com/docs/web-servers/lamp/how-to-install-a-lamp-stack-on-ubuntu-18-04/)
-   [mediaWiki Ubuntu](https://www.linode.com/docs//websites/wikis/install-mediawiki-on-ubuntu-1804/)
-   [Configuración MySQL](https://www.youtube.com/watch?v=JXDuVypcHNA)
-   [PostgreSQL](https://linuxhint.com/setup_postgresql_replication/)

