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
     ansible_ssh_private_key_file = "/home/challeger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "ansible_host" "puppetdb" {
  name = openstack_compute_instance_v2.tf_puppetdb.access_ip_v4
  groups = ["puppetdb"]
  variables = {
     ansible_user = "ubuntu" ,
     ansible_ssh_private_key_file = "/home/challeger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "ansible_host" "puppetagents" {
  count = var.puppetagentcount
  name = openstack_compute_instance_v2.tf_puppetagents[count.index].access_ip_v4
  groups = ["puppetagents"]
  variables = {
     ansible_user = "ubuntu" ,
     ansible_ssh_private_key_file = "/home/challeger-16/mykey.PEM",
  }
  depends_on = [time_sleep.wait_20_seconds]
}

resource "terraform_data" "ansible_inventory" {
  provisioner "local-exec" {
    command = "ansible-inventory -i /home/challenger-16/terraform-dir/inventory.yml --graph"
    }
  depends_on = [ansible_host.puppetserver ]
}

#resource "terraform_data" "ansible_playbook" {
 #  provisioner "local-exec" {
  #    command = "ansible-playbook -i /home/challenger-16/ansible-dir/myinventory.ini /home/challenger-16/ansible-dir/myplay.yml"
#      }
#   depends_on = [terraform_data.ansible_inventory]
#}
' > ansible.tf
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

Ejecutamos "terraform apply"

```
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform apply
openstack_compute_secgroup_v2.tf_puppetdb_sg: Refreshing state... [id=12073ba4-a5b3-4872-a3ba-4a9a7d236f1c]
openstack_compute_secgroup_v2.tf_puppetagents_sg: Refreshing state... [id=87155be4-2e5a-448e-ac95-91105c1f5d7d]
openstack_blockstorage_volume_v3.tf_puppetagents_vol[0]: Refreshing state... [id=d55b6118-f5d7-4e4b-a222-c45c84a7323a]
openstack_blockstorage_volume_v3.tf_puppetserver_vol: Refreshing state... [id=25e39d6f-f2af-433f-b98a-0f0c6eb37c84]
openstack_blockstorage_volume_v3.tf_puppetdb_vol: Refreshing state... [id=6cb520de-5b00-4af7-b40d-544222f60cce]
openstack_compute_secgroup_v2.tf_puppetserver_sg: Refreshing state... [id=54fc883a-fde8-452d-bcf3-524697e937fb]
openstack_compute_instance_v2.tf_puppetserver: Refreshing state... [id=3f084247-ca12-4e5d-8c98-eb84299ec809]
openstack_compute_instance_v2.tf_puppetagents[0]: Refreshing state... [id=5a57ac29-0757-4fc1-8816-2c4d94cc7b04]
openstack_compute_instance_v2.tf_puppetdb: Refreshing state... [id=8f260cc7-26b4-4bd6-9151-ba1d3712ed70]
time_sleep.wait_20_seconds: Refreshing state... [id=2024-07-10T00:18:11Z]
ansible_host.puppetdb: Refreshing state... [id=10.100.67.48]
ansible_host.puppetserver: Refreshing state... [id=10.100.65.79]
terraform_data.ansible_inventory: Refreshing state... [id=eaf53871-709b-8914-a417-0c8ab24812bd]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # ansible_host.puppetagents[0] will be created
  + resource "ansible_host" "puppetagents" {
      + groups    = [
          + "puppetagents",
        ]
      + id        = (known after apply)
      + name      = "10.100.67.13"
      + variables = {
          + "ansible_ssh_private_key_file" = "/home/challeger-16/mykey.PEM"
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
