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

## EJECUTAR MODULO DE TERRAFORM PARA DESPLEGAR INFRAESTRUCTURA Y EJECUTAR ANSIBLE PLAYBOOKS




