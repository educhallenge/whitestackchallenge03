# CHALLENGE 03  PASO 1: DESPLEGAR PUPPET EN AMBIENTE DE PRUEBAS


## LEVANTAMOS INSTANCIAS EN OPENSTACK

Desde nuestro Ubuntu remoto nos conectamos a Openstack para crear el keypair para SSH. Exportamos la llave privada al archivo "mykey.PEM" la cual luego usaremos para iniciar conexión SSH desde nuestro Ubuntu remoto hacia cada instancia que crearemos en Openstack.  Cambiamos los permisos para que "mykey.PEM" sea de solo lectura

```
challenger-16@challenge-3-pivote:~$ openstack keypair create mykey > mykey.PEM

challenger-16@challenge-3-pivote:~$ chmod 400 mykey.PEM

```

Creamos Security Group para Puppet Server

```

challenger-16@challenge-3-pivote:~$ openstack security group create --description puppetserver puppetserver_sg

challenger-16@challenge-3-pivote:~$ openstack security group rule create --proto tcp --dst-port 22 puppetserver_sg
challenger-16@challenge-3-pivote:~$ openstack security group rule create --proto tcp --dst-port 8140 puppetserver_sg
```

Creamos Security Group para Puppet Agent

```
challenger-16@challenge-3-pivote:~$ openstack security group create --description puppetagents puppetagents_sg

challenger-16@challenge-3-pivote:~$ openstack security group rule create --proto tcp --dst-port 22 puppetagents_sg
```

Creamos Security Group para Puppet DB
```
challenger-16@challenge-3-pivote:~$ openstack security group create --description puppetdb puppetdb_sg

challenger-16@challenge-3-pivote:~$ openstack security group rule create --proto tcp --dst-port 8081 puppetdb_sg
```

Creamos tres instancias: 1 PuppetServer,  1 Puppet Agent y 1 PuppetDB. Sólo usamos la network PUBLIC debido a que en las pruebas que hice fue la única network a la cual tenía acceso remoto mediante SSH. 

Notar también que a PuppetDB se le asigna un security group llamado "puppetagents_sg" y otro llamado "puppetdb_sg". Esto es porque PuppetDB también tiene un PuppetAgent instalado.

```
challenger-16@challenge-3-pivote:~$ openstack server create --flavor m1.small --image 8937e160-00b4-4327-acdf-41f2a58bb38f --key-name mykey  --security-group  puppetserver_sg --network PUBLIC step1_puppetserver

challenger-16@challenge-3-pivote:~$ openstack server create --flavor m1.small --image 8937e160-00b4-4327-acdf-41f2a58bb38f --key-name mykey  --security-group  puppetagents_sg --network PUBLIC step1_puppetagent1

challenger-16@challenge-3-pivote:~$ openstack server create --flavor m1.small --image 8937e160-00b4-4327-acdf-41f2a58bb38f --key-name mykey  --security-group  puppetagents_sg --security-group puppetdb_sg --network PUBLIC step1_puppetdb
```

Listamos los servidores y anotamos las direcciones IP que nos servirá en los siguientes pasos

```
challenger-16@challenge-3-pivote:~$ openstack server list
```


## DESPLEGAR PUPPETSERVER

Desde nuestro Ubuntu remoto nos conectamos a PuppetServer

```
challenger-16@challenge-3-pivote:~$  ssh ubuntu@<ip-de-PuppetServer> -i mykey.PEM
```

Editamos el archivo host para facilitar la comunicación entre los diferentes componentes de Puppet
```
step1-puppetserver:~$ sudo tee -a /etc/hosts >/dev/null <<'EOF'
<ip-de-PuppetServer> puppetserver puppet step1-puppetserver.openstacklocal
<ip-de-PuppetDB> step1-puppetdb.openstacklocal
EOF
```

Configurar el repositorio de Puppet

```
step1-puppetserver:~$ wget https://apt.puppet.com/puppet8-release-jammy.deb
step1-puppetserver:~$ sudo dpkg -i puppet8-release-jammy.deb
```

Instalar Puppet Server
```
step1-puppetserver:~$ sudo apt update
step1-puppetserver:~$ sudo apt install puppetserver -y
```

Editar el archivo de configuración de Puppet Server para que sólo use 512 MB de memoria en vez de los 2 GB que usa por defecto
```
step1-puppetserver:~$ sudo sed -i 's/Xms2g/Xms512m/' /etc/default/puppetserver
step1-puppetserver:~$ sudo sed -i 's/Xmx2g/Xmx512m/' /etc/default/puppetserver
```

Iniciar el servicio de Puppet Server
```
step1-puppetserver:~$ sudo systemctl start puppetserver
step1-puppetserver:~$ sudo systemctl enable puppetserver
step1-puppetserver:~$ sudo systemctl status puppetserver
```

Hacemos que bash cargue las nuevas variables de entorno

```
step1-puppetserver:~$ bash -l 
```

Verificamos la versión de Puppet Server
```
step1-puppetserver:~$ puppetserver -v
puppetserver version: 8.6.1
```

## DESPLEGAR PUPPET AGENTS

Desde nuestro Ubuntu remoto nos conectamos a cada Puppet Agent

```
challenger-16@challenge-3-pivote:~$  ssh ubuntu@<ip-de-PuppetAgent> -i mykey.PEM
```

Editamos el archivo host para facilitar la comunicación entre los diferentes componentes de Puppet
```
step1-puppetagent1:~$ sudo tee -a /etc/hosts >/dev/null <<'EOF'
<ip-de-PuppetServer> puppetserver puppet step1-puppetserver.openstacklocal
<ip-de-PuppetDB> step1-puppetdb.openstacklocal
EOF
```

Configurar el repositorio de Puppet

```
step1-puppetagent1:~$ wget https://apt.puppet.com/puppet8-release-jammy.deb
step1-puppetagent1:~$ sudo dpkg -i puppet8-release-jammy.deb
```

Instalar Puppet Agent
```
step1-puppetagent1:~$ sudo apt update
step1-puppetagent1:~$ sudo apt install puppet-agent -y
```

En mis pruebas vi varios popups con warnings respecto a la versión de kernel y servicios que se deben reinciar cuando se inicia el servicio de Puppet Agent. Dichos popups pueden romper los scripts de automatización que usaremos después, por ellos desactivamos dichos popups modificando el archivo "needrestart.conf" de la siguiente manera

```
step1-puppetagent1:~$ sudo cp /etc/needrestart/needrestart.conf /etc/needrestart/needrestart.conf.OLD
step1-puppetagent1:~$ sudo sed -i "s/#\$nrconf{kernelhints} = -1;/\$nrconf{kernelhints} = -1;/g" /etc/needrestart/needrestart.conf
step1-puppetagent1:~$ sudo sed -i "/#\$nrconf{restart} = 'i';/s/.*/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf
```

Iniciamos el servicio de Puppet Agent
```
step1-puppetagent1:~$ sudo systemctl start puppet
step1-puppetagent1:~$ sudo systemctl enable puppet
step1-puppetagent1:~$ sudo systemctl status puppet
```

Hacemos que bash cargue las nuevas variables de entorno
```
bash -l 
```

## PROCEDIMIENTO PARA COMUNICAR PUPPET AGENT CON PUPPET SERVER

 
En Puppet Agent ejecutamos el siguiente comando para probar la comunicación entre PuppetAgent y Puppet Server
```
 step1-puppetagent1:~$sudo /opt/puppetlabs/bin/puppet agent --test
```


En Puppet Server  verificamos que Puppet Agent ha pedido un certificado

```
step1-puppetserver:~$ sudo /opt/puppetlabs/bin/puppetserver ca list
Requested Certificates:
    step1-puppetagent1.openstacklocal       (SHA256)  04:DE:21:0D:F7:DB:10:9B:84:00:BA:82:33:BB:AB:E5:C0:ED:5A:3C:F4:E5:41:C9:3C:9F:06:B5:D0:F6:CC:F2
```

En Puppet Server  firmamos el certificado que nos pidió Puppet Agent

```
step1-puppetserver:~$  sudo /opt/puppetlabs/bin/puppetserver ca sign --all
Successfully signed certificate request for step1-puppetagent1.openstacklocal
```


En Puppet Agent volvemos a ejecutar el comando para hacer el test y ahora debe decir que todo fue exitoso
```
 step1-puppetagent1:~$sudo /opt/puppetlabs/bin/puppet agent --test
```


## DESPLEGAR PUPPETDB (parte1: desplegar PuppetAgent )

Para desplegar PuppetDB es prerequisito instalar PuppetAgent. Por eso en esta primera parte debemos ejecutar exactamente el mismo procedimiento que mencionamos para desplegar un Puppet Agent.


## DESPLEGAR PUPPETDB (parte2: despliegue desde PuppetServer)

En PuppetServer instalamos el módulo PuppetDB

```
ubuntu@step1-puppetserver:~$ sudo /opt/puppetlabs/bin/puppet  module install puppetlabs-puppetdb --version 8.1.0
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-puppetdb (v8.1.0)
  ├── puppetlabs-firewall (v8.0.2)
  ├─┬ puppetlabs-inifile (v6.1.1)
  │ └── puppetlabs-stdlib (v9.6.0)
  └─┬ puppetlabs-postgresql (v10.3.0)
    ├── puppet-systemd (v7.1.0)
    ├── puppetlabs-apt (v9.4.0)
    └── puppetlabs-concat (v9.0.2)
```

En PuppetServer creamos un manifiesto llamado "site.pp" para desplegar PuppetDB y PostgreSQL en la instancia PuppetDB  y también para configurar PuppetServer para que tenga comunicación con PuppetDB

```
step1-puppetserver:~$ cd /etc/puppetlabs/code/environments/production/manifests/

step1-puppetserver:~$ sudo tee -a site.pp >/dev/null <<'EOF'
node 'step1-puppetserver.openstacklocal' {
  # Here we configure the Puppet master to use PuppetDB,
  # telling it the hostname of the PuppetDB node
  class { 'puppetdb::master::config':
    puppetdb_server => 'step1-puppetdb.openstacklocal',
  }
}

node 'step1-puppetdb.openstacklocal' {
  class { 'puppetdb':
    database_host => 'step1-puppetdb.openstacklocal',
    database_listen_address => '0.0.0.0'
  }
} 
EOF

```

Desplegamos el manifiesto "site.pp" en Puppet Server

```
step1-puppetserver:/etc/puppetlabs/code/environments/production/manifests$ sudo /opt/puppetlabs/bin/puppet apply site.pp
```

Desplegamos el manifiesto "site.pp" en Puppet DB. El Test debe indicar un resultado exitoso

```
ubuntu@step1-puppetdb:~$ sudo /opt/puppetlabs/bin/puppet agent --test
```

