# CHALLENGE 03  PASO 2:  CREAR MODULO DE TERRAFORM


En el Ubuntu remoto ya tenemos instalado el comando para usar Terraform. Creamos un directorio para los scripts de Terraform
```
challenger-16@challenge-3-pivote:~$ mkdir terraform-dir
challenger-16@challenge-3-pivote:~$ cd terraform-dir/
```

Creamos el archivo "provider.tf" para que Terraform se  pueda conectar a Openstack

```
challenger-16@challenge-3-pivote:~/terraform-dir$  echo '
terraform {
  required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.53.0"
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

Creamos el archivo "variables.tf" para definir cuántas instancias de Puppet Agents se van a crear

```
challenger-16@challenge-3-pivote:~/terraform-dir$  echo '
variable "puppetagentcount" {
 type        = number
 default     = 1
}
' > variables.tf
```

Creamos el archivo "securitygroups.tf" para definir los Security Groups

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo '
resource "openstack_compute_secgroup_v2" "tf_puppetserver_sg" {
  name        = "tf_puppetserver_sg"
  description = "terraform puppet server sg"

  rule {
    from_port   = 22
    to_port     = 22
    ip_protocol = "tcp"
    cidr        = "0.0.0.0/0"
  }

  rule {
    from_port   = 8140
    to_port     = 8140
    ip_protocol = "tcp"
    cidr        = "0.0.0.0/0"
  }
}

resource "openstack_compute_secgroup_v2" "tf_puppetagents_sg" {
  name        = "tf_puppetagents_sg"
  description = "terraform puppet agents sg"

  rule {
    from_port   = 22
    to_port     = 22
    ip_protocol = "tcp"
    cidr        = "0.0.0.0/0"
  }
}

resource "openstack_compute_secgroup_v2" "tf_puppetdb_sg" {
  name        = "tf_puppetdb_sg"
  description = "terraform puppet db sg"

  rule {
    from_port   = 8081
    to_port     = 8081
    ip_protocol = "tcp"
    cidr        = "0.0.0.0/0"
  }
}
' > securitygroups.tf
```

Creamos el archivo "volumes.tf" para definir los volumes que después se usarán en la creación de instancias. Notar que se usa la variable count que a su vez referencia a la variable puppetagentcount para definir el número de volumes para los Puppet Agents.

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo '
resource "openstack_blockstorage_volume_v3" "tf_puppetserver_vol" {
  name = "tf_puppeserver_vol"
  size = 20
  image_id = "8937e160-00b4-4327-acdf-41f2a58bb38f"
}

resource "openstack_blockstorage_volume_v3" "tf_puppetagents_vol" {
  name = "tf_puppetagents_vol_${count.index}"
  size = 20
  image_id = "8937e160-00b4-4327-acdf-41f2a58bb38f"
  count = var.puppetagentcount
}

resource "openstack_blockstorage_volume_v3" "tf_puppetdb_vol" {
  name = "tf_puppetdb_vol"
  size = 20
  image_id = "8937e160-00b4-4327-acdf-41f2a58bb38f"
}
' > volumes.tf
```


Creamos el archivo "compute.tf" para definir las instancias que se van a crear. Notar que se usa la variable count que a su vez referencia a la variable puppetagentcount para definir el número de instancias de Puppet Agents. En el caso de PuppetServer sólo se creará una instancia. Igual para PuppetDB sólo se creará una instancia

```
challenger-16@challenge-3-pivote:~/terraform-dir$ echo '
resource "openstack_compute_instance_v2" "tf_puppetserver" {
  name            = "tf_puppetserver"
  flavor_id       = "m1.small"
  key_pair        = "mykey"
  security_groups = ["tf_puppetserver_sg"]

  block_device {
    uuid = "${openstack_blockstorage_volume_v3.tf_puppetserver_vol.id}"
    source_type = "volume"
    boot_index = 0
    volume_size = "${openstack_blockstorage_volume_v3.tf_puppetserver_vol.size}"
    destination_type = "volume"
    delete_on_termination = true
  }

  network {
    name = "PUBLIC"
  }
}

resource "openstack_compute_instance_v2" "tf_puppetagents" {
  count =  var.puppetagentcount
  name            = "tf_puppetagent_${count.index}"
  flavor_id       = "m1.small"
  key_pair        = "mykey"
  security_groups = ["tf_puppetagents_sg"]

  block_device {
    uuid = "${openstack_blockstorage_volume_v3.tf_puppetagents_vol[count.index].id}"
    source_type = "volume"
    boot_index = 0
    volume_size = "${openstack_blockstorage_volume_v3.tf_puppetagents_vol[count.index].size}"
    destination_type = "volume"
    delete_on_termination = true
  }

  network {
    name = "PUBLIC"
  }
}

resource "openstack_compute_instance_v2" "tf_puppetdb" {
  name            = "tf_puppetdb"
  flavor_id       = "m1.small"
  key_pair        = "mykey"
  security_groups = ["tf_puppetdb_sg","tf_puppetagents_sg"]

  block_device {
    uuid = "${openstack_blockstorage_volume_v3.tf_puppetdb_vol.id}"
    source_type = "volume"
    boot_index = 0
    volume_size = "${openstack_blockstorage_volume_v3.tf_puppetdb_vol.size}"
    destination_type = "volume"
    delete_on_termination = true
  }

  network {
    name = "PUBLIC"
  }
}
' > compute.tf

```