# Oracle Database 12c (Centos)

**DEPRECATED: Use the ansible version instead https://github.com/albmtez/ansible-oracle-12c**

https://www.howtoforge.com/tutorial/how-to-install-oracle-database-12c-on-centos-7/

Vamos a hacer la instalación usando Vagrant y el box **albmtez/centos7-x64-m**.

## Configuración

Instalación de los paquetes necesarios por Oracle:

```
# yum install -y binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 \
glibc.i686 glibc.x86_64 glibc-devel.i686 glibc-devel.x86_64 ksh \
compat-libstdc++-33 libaio.i686 libaio.x86_64 libaio-devel.i686 \
libaio-devel.x86_64 libgcc.i686 libgcc.x86_64 libstdc++.i686 \
libstdc++.x86_64 libstdc++-devel.i686 libstdc++-devel.x86_64 libXi.i686 \
libXi.x86_64 libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64 \
smartmontools.x86_64
```

Creación de grupos y usuario:

```
# groupadd oinstall
# groupadd dba
# useradd -g oinstall -G dba oracle
# passwd oracle
```

Ajustamos la configuración del sistema. Editamos el archivo /etc/sysctl.conf y añadimos lo siguiente:

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 2147483648
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```

Para hacer efectivos los cambios, ejecutamos los siguientes comandos:

```
# sysctl -p
# sysctl -a
```

Editamos el archivo /etc/security/limits.conf y añadimos:

```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 32768
oracle hard memlock 134217728
oracle soft memlock 134217728
```

Instalamos X Window System para poder lanzar el asistente de instalación de Oracle de forma remota, así como los paquetes necesarios xauth y xdpyinfo:

```
# yum install -y xauth xdpyinfo
```

Habilitamos el X11 forwarding añadiendo las líneas siguientes al archivo /etc/ssh/sshd_config:

```
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
```

Reiniciamos el servicio sshd:

```
# service sshd restart
```

Creamos la estructura de directorios:

```
# mkdir -p /opt/oracle/product/12.1.0.2
# mkdir -p /opt/oraInventory
# chown -R oracle:dba /opt/oracle/
# chown -R oracle:dba /opt/oraInventory
```

Añadimos las variables de entorno siguientes al usuario Oracle:

```
# echo "export ORACLE_HOSTNAME=localhost" >> /home/oracle/.bashrc
# echo "export ORACLE_OWNER=oracle" >> /home/oracle/.bashrc
# echo "export ORACLE_BASE=/opt/oracle" >> /home/oracle/.bashrc
# echo "export ORACLE_HOME=/opt/oracle/product/12.1.0.2/dbhome_1" >> /home/oracle/.bashrc
# echo "export ORACLE_HOME_LISTNER=$ORACLE_HOME" >> /home/oracle/.bashrc
# echo "export ORACLE_UNQNAME=orcl" >> /home/oracle/.bashrc
# echo "export ORACLE_SID=orcl" >> /home/oracle/.bashrc
# echo "export PATH=\$PATH:\$ORACLE_HOME/bin" >> /home/oracle/.bashrc
# echo "export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib" >> /home/oracle/.bashrc
# echo "export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib" >> /home/oracle/.bashrc
```

Deshabilitamos el cortafuegos:

```
# systemctl stop firewalld
# systemctl disable firewalld
```

### Vagrantfile

Se puede usar el archivo *Vagrantfile* para levantar y aprovisionar la máquina con toda esta configuración.

## Instalación

Descomprimimos el instalador:

```
# unzip /vagrant/linuxx64_12201_database.zip -d /home/oracle
# chown -R oracle:oinstall /home/oracle/database
```

Nos conectamos con el usuario oracle:

```
$ ssh -X oracle@host_oracle
```

Y ejecutamos el instalador:

```
$ cd /home/oracle/database
$ ./runInstaller
```

## Post-instalación

Arrancamos la base de datos y el listener con:

```
$ dbstart $ORACLE_HOME_LISTNER
```

Paramos la base de datos y el listener con:

```
$ dbshut $ORACLE_HOME_LISTNER
```

Editamos el archivo */u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/listener.ora* para indicar el puerto *1521*:

```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    )
  )
```

Borramos el instalador:

```
$ rm -rf /home/oracle/database
```

## Arranque automático

https://docs.oracle.com/database/121/UNXAR/strt_stp.htm#UNXAR417

Editamos el archivo **/etc/oratab** y nos vamos a la última línea para cambiar el último valor por “Y” para que la base de datos se inicie de forma automática al arrancar el sistema:

```
orcl:/u01/app/oracle/product/12.1.0.2/dbhome_1:Y
```

Copiamos el script de control **dbora** en **/etc/init.d**.

Cambiamos el grupo y le damos permisos:

```
# chgrp dba dbora
# chmod 750 dbora
```

Y creamos los enlaces:

```
# ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
# ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
# ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
```

## Oracle Enterprise Manager Database Express

Oracle Enterprise Manager Database Express accesible en: https://localhost:5500/em
