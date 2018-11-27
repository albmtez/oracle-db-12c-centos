# -*- mode: ruby -*-
# vi: set ft=ruby :

$init_oracle = <<SCRIPT
# Actualización de paquetes.
yum update -y

# Instalación de los paquetes necesarios.
yum install -y binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.i686 \
glibc.x86_64 glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 libaio.i686 \
libaio.x86_64 libaio-devel.i686 libaio-devel.x86_64 libgcc.i686 libgcc.x86_64 libstdc++.i686 \
libstdc++.x86_64 libstdc++-devel.i686 libstdc++-devel.x86_64 libXi.i686 libXi.x86_64 \
libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64 smartmontools.x86_64

# Creación de grupos y usuario.
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba oracle
echo "oracle*" | passwd oracle --stdin

# Actualización del archivo /etc/sysctl.conf
echo "\n\n# Oracle 12c" >> /etc/sysctl.conf
echo "fs.aio-max-nr = 1048576" >> /etc/sysctl.conf
echo "fs.file-max = 6815744" >> /etc/sysctl.conf
echo "kernel.shmall = 2097152" >> /etc/sysctl.conf
echo "kernel.shmmax = 2147483648" >> /etc/sysctl.conf
echo "kernel.shmmni = 4096" >> /etc/sysctl.conf
echo "kernel.sem = 250 32000 100 128" >> /etc/sysctl.conf
echo "net.ipv4.ip_local_port_range = 9000 65500" >> /etc/sysctl.conf
echo "net.core.rmem_default = 262144" >> /etc/sysctl.conf
echo "net.core.rmem_max = 4194304" >> /etc/sysctl.conf
echo "net.core.wmem_default = 262144" >> /etc/sysctl.conf
echo "net.core.wmem_max = 1048586" >> /etc/sysctl.conf

sysctl -p
sysctl -a

# Modificación de limits para el usuario oracle.
echo "\n\n# Oracle 12c" >> /etc/security/limits.conf
echo "oracle soft nproc 2047" >> /etc/security/limits.conf
echo "oracle hard nproc 16384" >> /etc/security/limits.conf
echo "oracle soft nofile 1024" >> /etc/security/limits.conf
echo "oracle hard nofile 65536" >> /etc/security/limits.conf
echo "oracle soft stack 10240" >> /etc/security/limits.conf
echo "oracle hard stack 32768" >> /etc/security/limits.conf
echo "oracle hard memlock 134217728" >> /etc/security/limits.conf
echo "oracle soft memlock 134217728" >> /etc/security/limits.conf

# Creamos la estructura de directorios.
mkdir -p /opt/oracle/product/12.1.0.2
mkdir -p /opt/oraInventory
chown -R oracle:dba /opt/oracle/
chown -R oracle:dba /opt/oraInventory

# Instalamos los paquetes necesarios para ejecutar el OUI con X11 forwarding.
yum install -y xauth xdpyinfo

# Habilidando el X11 forwarding en ssh.
echo "\n\n# Enabling X11 forwarding" >> /etc/ssh/sshd_config
echo "X11Forwarding yes" >> /etc/ssh/sshd_config
echo "X11DisplayOffset 10" >> /etc/ssh/sshd_config
echo "X11UseLocalhost yes" >> /etc/ssh/sshd_config
service sshd restart

# Establecemos las variables de entorno para el usuario oracle.
echo "export ORACLE_HOSTNAME=localhost" >> /home/oracle/.bashrc
echo "export ORACLE_OWNER=oracle" >> /home/oracle/.bashrc
echo "export ORACLE_BASE=/opt/oracle" >> /home/oracle/.bashrc
echo "export ORACLE_HOME=/opt/oracle/product/12.1.0.2/dbhome_1" >> /home/oracle/.bashrc
echo "export ORACLE_HOME_LISTNER=$ORACLE_HOME" >> /home/oracle/.bashrc
echo "export ORACLE_UNQNAME=orcl" >> /home/oracle/.bashrc
echo "export ORACLE_SID=orcl" >> /home/oracle/.bashrc
echo "export PATH=\\$PATH:\\$ORACLE_HOME/bin" >> /home/oracle/.bashrc
echo “export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib” >> /home/oracle/.bashrc
echo “export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib” >> /home/oracle/.bashrc

# Deshabilitamos el cortafuegos
systemctl stop firewalld
systemctl disable firewalld
SCRIPT

Vagrant.configure("2") do |config|
	config.vm.box = "albmtez/centos7-x64-m"
	config.vm.define "Oracle12c"
	config.vm.hostname = "oracle"
	#config.disksize.size = '40GB'
	config.vm.network "private_network", ip: "10.0.0.10"
	config.vm.provision "shell", inline: $init_oracle
	config.ssh.insert_key = false
	config.vm.provider "virtualbox" do |vb|
		# Display the VirtualBox GUI when booting the machine
		vb.gui = false
		vb.name = "Oracle12c"
		# Customize the amount of memory on the VM:
		#vb.memory = 4096
		#vb.cpus = 2
	end
end