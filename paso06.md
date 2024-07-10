# CHALLENGE 03: PASO 6


## ARCHIVOS DE TERRAFORM

Vamos al directorio "terraform-dir" que usamos en los pasos 2 y 3.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ ls -hal *tf
-rw-rw-r-- 1 challenger-16 challenger-16 1.7K Jul  9 02:31 compute.tf
-rw-rw-r-- 1 challenger-16 challenger-16  389 Jul  9 00:14 provider.tf
-rw-rw-r-- 1 challenger-16 challenger-16  884 Jul  9 02:19 securitygroups.tf
-rw-rw-r-- 1 challenger-16 challenger-16   71 Jul  9 02:16 variables.tf
-rw-rw-r-- 1 challenger-16 challenger-16  536 Jul  9 02:31 volumes.tf
```
Reusaremos todos los archivos *.tf.  La única excepción es el archivo "provider.tf" al cual le agregaremos un provider más para poder conectarse a Ansible.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo ' terraform {
  required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.53.0"
    }
    ansible = {
      source = "ansible/ansible"
      version = "1.3.0"
    }
  }
}

provider "openstack" {
  user_name   = "challenger-16"
  tenant_name = "challenger-16"
  password    = "oKah9uos4jae4keebahb\\"
  auth_url    = "http://10.100.1.31:5000/v3"
  region      = "RegionOne"
}
' > provider.tf
```


Verificamos con "terraform init"

```
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform init

Initializing the backend...

Initializing provider plugins...
- terraform.io/builtin/terraform is built in to Terraform
- Finding ansible/ansible versions matching "1.3.0"...
- Reusing previous version of terraform-provider-openstack/openstack from the dependency lock file
- Installing ansible/ansible v1.3.0...
- Installed ansible/ansible v1.3.0 (self-signed, key ID 41F01D0480007165)
- Using previously-installed terraform-provider-openstack/openstack v1.53.0

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!
```


También vamos a crear un archivo "ansibleinventory.tf" para crear un inventario dinámico para Ansible.
```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo '
resource "time_sleep" "wait_20_seconds" {
  depends_on = [openstack_compute_instance_v2.tf_puppetserver, openstack_compute_instance_v2.tf_puppetagents, openstack_compute_instance_v2.tf_puppetdb ]
  create_duration = "20s"
}

resource "ansible_host" "puppetserver" {
  name = openstack_compute_instance_v2.tf_puppetserver.access_ip_v4
  groups = ["puppetserver"]
  variables = {
     ansible_user = "ubuntu" ,
     ansible_ssh_private_key_file = "/home/challenger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "ansible_host" "puppetdb" {
  name = openstack_compute_instance_v2.tf_puppetdb.access_ip_v4
  groups = ["puppetdb"]
  variables = {
     ansible_user = "ubuntu" ,
     ansible_ssh_private_key_file = "/home/challenger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "ansible_host" "puppetagents" {
  count = var.puppetagentcount
  name = openstack_compute_instance_v2.tf_puppetagents[count.index].access_ip_v4
  groups = ["puppetagents"]
  variables = {
     ansible_user = "ubuntu" ,
     ansible_ssh_private_key_file = "/home/challenger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "terraform_data" "ansible_inventory" {
  provisioner "local-exec" {
    command = "ansible-inventory -i /home/challenger-16/terraform-dir/inventory.yml --graph"
    }
  depends_on = [ansible_host.puppetserver ]
}
' > ansibleinventory.tf
```

## ARCHIVOS DE ANSIBLE

Instalar el plugin de Terraform para Ansible
```
challenger-16@challenge-3-pivote:~/terraform-dir$ ansible-galaxy collection install cloud.terraform
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/cloud-terraform-3.0.0.tar.gz to /home/challenger-16/.ansible/tmp/ansible-local-3344357gtlzzxx/tmpjvlazxh2/cloud-terraform-3.0.0-9wypcrte
Installing 'cloud.terraform:3.0.0' to '/home/challenger-16/.ansible/collections/ansible_collections/cloud/terraform'
cloud.terraform:3.0.0 was installed successfully
```

Crear archivo con inventario de Ansible y usar el plugin instalado. Dicho plugin luego recibirá dinámicante la información de las direcciones IP de las instancias que Terraform creará.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo 'plugin: cloud.terraform.terraform_provider' > inventory.yml
challenger-16@challenge-3-pivote:~/terraform-dir$
```

## EJECUTAR MODULO DE TERRAFORM PARA DESPLEGAR INFRAESTRUCTURA Y GENERAR INVENTORY PARA ANSIBLE

Ejecutamos "terraform apply"

```
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform apply
### output omitido por brevedad

  # ansible_host.puppetagents[0] will be created
  + resource "ansible_host" "puppetagents" {
      + groups    = [
          + "puppetagents",
        ]
      + id        = (known after apply)
      + name      = "10.100.67.13"
      + variables = {
          + "ansible_ssh_private_key_file" = "/home/challenger-16/mykey.PEM"
          + "ansible_user"                 = "ubuntu"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

ansible_host.puppetagents[0]: Creating...
ansible_host.puppetagents[0]: Creation complete after 0s [id=10.100.67.13]
```

Verificamos que el inventario de Ansible fue generado dinámicamente.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ ansible-inventory -i /home/challenger-16/terraform-dir/inventory.yml --graph
@all:
  |--@ungrouped:
  |--@puppetagents:
  |  |--10.100.67.13
  |--@puppetdb:
  |  |--10.100.67.48
  |--@puppetserver:
  |  |--10.100.65.79
```

Lo comparamos con "openstack server list" y vemos que las IPs en el inventario de Ansible son las correctas

```
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack server list
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| ID                                   | Name             | Status | Networks            | Image                    | Flavor   |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| 3f084247-ca12-4e5d-8c98-eb84299ec809 | tf_puppetserver  | ACTIVE | PUBLIC=10.100.65.79 | N/A (booted from volume) | m1.small |
| 5a57ac29-0757-4fc1-8816-2c4d94cc7b04 | tf_puppetagent_0 | ACTIVE | PUBLIC=10.100.67.13 | N/A (booted from volume) | m1.small |
| 8f260cc7-26b4-4bd6-9151-ba1d3712ed70 | tf_puppetdb      | ACTIVE | PUBLIC=10.100.67.48 | N/A (booted from volume) | m1.small |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
```

## AGREGAR MODULO DE TERRAFORM PARA EJECUTAR PLAYBOOK DE ANSIBLE

Vamos a reutilizar el playbook "home/challenger-16/ansible-dir/myplay.yml" que habíamos creado en el paso 4.  Dicho playbook será ejecutado por el módulo "ansibleplay.tf" que mostramos a continuación.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo '
resource "terraform_data" "ansible_playbook" {
   provisioner "local-exec" {
      command = "ansible-playbook -i /home/challenger-16/terraform-dir/inventory.yml /home/challenger-16/ansible-dir/myplay.yml"
      }
   depends_on = [terraform_data.ansible_inventory]
}
' > ansibleplay.tf
```

Ahora ejecutamos "terraform apply" y en los logs vemos que el playbook se ejecuta correctamente

```
```
