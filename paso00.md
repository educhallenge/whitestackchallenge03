# CHALLENGE 03  PASO 0: PROBAR ACCESOS


En un Linux local con sistema operativo Ubuntu se copió el archivo "challenger-16" con la llave privada SSH y se modificó los permisos para restringir a sólo lectura

```
ubuntu@ubuntu:~$ ls -hal challenger-16
-rw-rw-r-- 1 ubuntu ubuntu 419 Jun 25 15:58 challenger-16

ubuntu@ubuntu:~$ chmod 400 challenger-16
ubuntu@ubuntu:~$ ls -hal challenger-16
-r-------- 1 ubuntu ubuntu 419 Jun 25 15:58 challenger-16
```

Desde el Linux local se ingresa a un Ubuntu remoto y al mismo tiepmo se aprovecha en hacer portforwarding para poder tener acceso a la interfaz gráfica de Openstack (10.100.1.31:80) 

En Ubuntu remoto tenemos el archivo "challenger-16-openrc.sh" que usamos para obtener las variables de entorno para conectarnos a Openstack

```
ubuntu@ubuntu:~$ ssh -p 3822  challenger-16@201.217.240.69  -i challenger-16 -L 0.0.0.0:8080:10.100.1.31:80

challenger-16@challenge-3-pivote:~$ ls -hal *sh
-rw-r--r-- 1 challenger-16 challenger-16 2.0K Jun  8 22:36 challenger-16-openrc.sh

challenger-16@challenge-3-pivote:~$ source challenger-16-openrc.sh
Please enter your OpenStack Password for project challenger-16 as user challenger-16:
```

Desde el Ubuntu remoto probamos que tenemos conexión a Openstack con el comando "openstack versions show"

```
challenger-16@challenge-3-pivote:~$ openstack versions show
+-------------+----------------+---------+-----------+-------------------------------+------------------+------------------+
| Region Name | Service Type   | Version | Status    | Endpoint                      | Min Microversion | Max Microversion |
+-------------+----------------+---------+-----------+-------------------------------+------------------+------------------+
| RegionOne   | block-storage  | 3.0     | CURRENT   | http://10.100.1.31:8776/v3/   | 3.0              | 3.70             |
| RegionOne   | identity       | 3.14    | CURRENT   | http://10.100.1.31:5000/v3/   | None             | None             |
| RegionOne   | network        | 2.0     | CURRENT   | http://10.100.1.31:9696/v2.0/ | None             | None             |
| RegionOne   | compute        | 2.0     | SUPPORTED | http://10.100.1.31:8774/v2/   | None             | None             |
| RegionOne   | compute        | 2.1     | CURRENT   | http://10.100.1.31:8774/v2.1/ | 2.1              | 2.93             |
| RegionOne   | cloudformation | 1.0     | CURRENT   | http://10.100.1.31:8000/v1/   | None             | None             |
| RegionOne   | image          | 2.0     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.1     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.2     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.3     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.4     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.5     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.6     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.7     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.8     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.9     | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.10    | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.11    | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.12    | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.13    | SUPPORTED | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | image          | 2.15    | CURRENT   | http://10.100.1.31:9292/v2/   | None             | None             |
| RegionOne   | orchestration  | 1.0     | CURRENT   | http://10.100.1.31:8004/v1/   | None             | None             |
| RegionOne   | placement      | 1.0     | CURRENT   | http://10.100.1.31:8780/      | 1.0              | 1.39             |
+-------------+----------------+---------+-----------+-------------------------------+------------------+------------------+
```