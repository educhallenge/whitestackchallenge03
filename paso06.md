# CHALLENGE 03: PASO 6

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
echo ' terraform {
  required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.53.0"
    }
    ansible = {
      source = "ansible/ansible"
      version = "1.1.0"
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


También vamos a crear un archivo "ansible.tf" para crear un inventario dinámico y para ejecutar el playbook de Ansible.
```
challenger-16@challenge-3-pivote:~/terraform-dir$ more ansible.tf
resource "terraform_data" "ansible_inventory" {
        provisioner "local-exec" {
      command = "ansible-inventory -i /home/challenger-16/ansible-dir/myinventory.ini --graph"
        }
    depends_on = [openstack_compute_instance_v2.tf_puppetserver, openstack_compute_instance_v2.tf_puppetagents, openstack_compute_instance_v2.tf_puppetdb ]
}

resource "terraform_data" "ansible_playbook" {
        provisioner "local-exec" {
      command = "ansible-playbook -i /home/challenger-16/ansible-dir/myinventory.ini /home/challenger-16/ansible-dir/myplay.yml"
        }
        depends_on = [terraform_data.ansible_inventory]
}
```


Crear archivo con inventario de Ansible. Dicho archivo sólo tendrá un plugin que luego recibirá dinámicante la información de los hostnames y direcciones IP de las instancias que Terraform creará.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo 'plugin: cloud.terraform.terraform_provider' > inventory.yml
challenger-16@challenge-3-pivote:~/terraform-dir$
```
