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
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform apply
openstack_compute_secgroup_v2.tf_puppetserver_sg: Refreshing state... [id=54fc883a-fde8-452d-bcf3-524697e937fb]
openstack_blockstorage_volume_v3.tf_puppetserver_vol: Refreshing state... [id=25e39d6f-f2af-433f-b98a-0f0c6eb37c84]
openstack_compute_secgroup_v2.tf_puppetdb_sg: Refreshing state... [id=12073ba4-a5b3-4872-a3ba-4a9a7d236f1c]
openstack_blockstorage_volume_v3.tf_puppetdb_vol: Refreshing state... [id=6cb520de-5b00-4af7-b40d-544222f60cce]
openstack_compute_secgroup_v2.tf_puppetagents_sg: Refreshing state... [id=87155be4-2e5a-448e-ac95-91105c1f5d7d]
openstack_blockstorage_volume_v3.tf_puppetagents_vol[0]: Refreshing state... [id=d55b6118-f5d7-4e4b-a222-c45c84a7323a]
openstack_compute_instance_v2.tf_puppetdb: Refreshing state... [id=8f260cc7-26b4-4bd6-9151-ba1d3712ed70]
openstack_compute_instance_v2.tf_puppetagents[0]: Refreshing state... [id=5a57ac29-0757-4fc1-8816-2c4d94cc7b04]
openstack_compute_instance_v2.tf_puppetserver: Refreshing state... [id=3f084247-ca12-4e5d-8c98-eb84299ec809]
time_sleep.wait_20_seconds: Refreshing state... [id=2024-07-10T00:18:11Z]
ansible_host.puppetdb: Refreshing state... [id=10.100.67.48]
ansible_host.puppetagents[0]: Refreshing state... [id=10.100.67.13]
ansible_host.puppetserver: Refreshing state... [id=10.100.65.79]
terraform_data.ansible_inventory: Refreshing state... [id=eaf53871-709b-8914-a417-0c8ab24812bd]
terraform_data.ansible_playbook: Refreshing state... [id=a1677348-8937-7cd2-3a85-1f5dabb52131]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # ansible_host.puppetagents[0] will be updated in-place
  ~ resource "ansible_host" "puppetagents" {
        id        = "10.100.67.13"
        name      = "10.100.67.13"
      ~ variables = {
          ~ "ansible_ssh_private_key_file" = "/home/challeger-16/mykey.PEM" -> "/home/challenger-16/mykey.PEM"
            # (1 unchanged element hidden)
        }
        # (1 unchanged attribute hidden)
    }

  # ansible_host.puppetdb will be updated in-place
  ~ resource "ansible_host" "puppetdb" {
        id        = "10.100.67.48"
        name      = "10.100.67.48"
      ~ variables = {
          ~ "ansible_ssh_private_key_file" = "/home/challeger-16/mykey.PEM" -> "/home/challenger-16/mykey.PEM"
            # (1 unchanged element hidden)
        }
        # (1 unchanged attribute hidden)
    }

  # ansible_host.puppetserver will be updated in-place
  ~ resource "ansible_host" "puppetserver" {
        id        = "10.100.65.79"
        name      = "10.100.65.79"
      ~ variables = {
          ~ "ansible_ssh_private_key_file" = "/home/challeger-16/mykey.PEM" -> "/home/challenger-16/mykey.PEM"
            # (1 unchanged element hidden)
        }
        # (1 unchanged attribute hidden)
    }

  # terraform_data.ansible_playbook is tainted, so must be replaced
-/+ resource "terraform_data" "ansible_playbook" {
      ~ id = "a1677348-8937-7cd2-3a85-1f5dabb52131" -> (known after apply)
    }

Plan: 1 to add, 3 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

terraform_data.ansible_playbook: Destroying... [id=a1677348-8937-7cd2-3a85-1f5dabb52131]
terraform_data.ansible_playbook: Destruction complete after 0s
ansible_host.puppetserver: Modifying... [id=10.100.65.79]
ansible_host.puppetdb: Modifying... [id=10.100.67.48]
ansible_host.puppetagents[0]: Modifying... [id=10.100.67.13]
ansible_host.puppetserver: Modifications complete after 0s [id=10.100.65.79]
terraform_data.ansible_playbook: Creating...
terraform_data.ansible_playbook: Provisioning with 'local-exec'...
terraform_data.ansible_playbook (local-exec): Executing: ["/bin/sh" "-c" "ansible-playbook -i /home/challenger-16/terraform-dir/inventory.yml /home/challenger-16/ansible-dir/myplay.yml"]
ansible_host.puppetdb: Modifications complete after 0s [id=10.100.67.48]
ansible_host.puppetagents[0]: Modifications complete after 0s [id=10.100.67.13]

terraform_data.ansible_playbook (local-exec): PLAY [all] *********************************************************************

terraform_data.ansible_playbook (local-exec): TASK [Edit /etc/hosts file] ****************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Download Puppet.deb] *****************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48]
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Install Puppet repository] ***********************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Apt update] **************************************************************
terraform_data.ansible_playbook: Still creating... [10s elapsed]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): PLAY [puppetagents puppetdb] ***************************************************

terraform_data.ansible_playbook (local-exec): TASK [Install Puppet Agent] ****************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Avoid kernel warning popups] *********************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Avoid services need to be restarted warning popups] **********************
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Start and enable services] ***********************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Check service status] ****************************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Show service status] *****************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "● puppet.service - Puppet agent",
terraform_data.ansible_playbook (local-exec):         "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
terraform_data.ansible_playbook (local-exec):         "     Active: active (running) since Tue 2024-07-09 08:06:55 UTC; 16h ago",
terraform_data.ansible_playbook (local-exec):         "       Docs: man:puppet-agent(8)",
terraform_data.ansible_playbook (local-exec):         "   Main PID: 8854 (puppet)",
terraform_data.ansible_playbook (local-exec):         "      Tasks: 2 (limit: 2324)",
terraform_data.ansible_playbook (local-exec):         "     Memory: 77.1M",
terraform_data.ansible_playbook (local-exec):         "        CPU: 2min 28.025s",
terraform_data.ansible_playbook (local-exec):         "     CGroup: /system.slice/puppet.service",
terraform_data.ansible_playbook (local-exec):         "             └─8854 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
terraform_data.ansible_playbook (local-exec):         "",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:07:02 tf-puppetagent-0 puppet-agent[32016]: Applied catalog in 0.01 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:37:02 tf-puppetagent-0 puppet-agent[32142]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:37:02 tf-puppetagent-0 puppet-agent[32142]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:37:02 tf-puppetagent-0 puppet-agent[32142]: Applied catalog in 0.01 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:07:02 tf-puppetagent-0 puppet-agent[32356]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:07:02 tf-puppetagent-0 puppet-agent[32356]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:07:02 tf-puppetagent-0 puppet-agent[32356]: Applied catalog in 0.02 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:37:02 tf-puppetagent-0 puppet-agent[32483]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:37:02 tf-puppetagent-0 puppet-agent[32483]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:37:02 tf-puppetagent-0 puppet-agent[32483]: Applied catalog in 0.01 seconds"
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "● puppet.service - Puppet agent",
terraform_data.ansible_playbook (local-exec):         "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
terraform_data.ansible_playbook (local-exec):         "     Active: active (running) since Tue 2024-07-09 08:39:50 UTC; 16h ago",
terraform_data.ansible_playbook (local-exec):         "       Docs: man:puppet-agent(8)",
terraform_data.ansible_playbook (local-exec):         "   Main PID: 10355 (puppet)",
terraform_data.ansible_playbook (local-exec):         "      Tasks: 2 (limit: 2324)",
terraform_data.ansible_playbook (local-exec):         "     Memory: 77.8M",
terraform_data.ansible_playbook (local-exec):         "        CPU: 3min 28.806s",
terraform_data.ansible_playbook (local-exec):         "     CGroup: /system.slice/puppet.service",
terraform_data.ansible_playbook (local-exec):         "             └─10355 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
terraform_data.ansible_playbook (local-exec):         "",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:10:00 tf-puppetdb puppet-agent[43969]: Applied catalog in 1.31 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:39:56 tf-puppetdb puppet-agent[44840]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:39:58 tf-puppetdb puppet-agent[44840]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 09 23:40:00 tf-puppetdb puppet-agent[44840]: Applied catalog in 1.26 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:09:56 tf-puppetdb puppet-agent[45310]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:09:59 tf-puppetdb puppet-agent[45310]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:10:00 tf-puppetdb puppet-agent[45310]: Applied catalog in 1.28 seconds",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:39:56 tf-puppetdb puppet-agent[45636]: Requesting catalog from puppet:8140 (10.100.65.79)",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:39:58 tf-puppetdb puppet-agent[45636]: Catalog compiled by tf-puppetserver.openstacklocal",
terraform_data.ansible_playbook (local-exec):         "Jul 10 00:40:00 tf-puppetdb puppet-agent[45636]: Applied catalog in 1.39 seconds"
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }

terraform_data.ansible_playbook (local-exec): PLAY [puppetserver] ************************************************************

terraform_data.ansible_playbook (local-exec): TASK [Install Puppet Server] ***************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Change Java options Xms to reduce memory usage] **************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Change Java Xmx to reduce memory usage] **********************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Start and enable services] ***********************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Check service status] ****************************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Show service status] *****************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "● puppetserver.service - puppetserver Service",
terraform_data.ansible_playbook (local-exec):         "     Loaded: loaded (/lib/systemd/system/puppetserver.service; enabled; vendor preset: enabled)",
terraform_data.ansible_playbook (local-exec):         "     Active: active (running) since Tue 2024-07-09 09:24:10 UTC; 15h ago",
terraform_data.ansible_playbook (local-exec):         "   Main PID: 25102 (java)",
terraform_data.ansible_playbook (local-exec):         "      Tasks: 54 (limit: 4915)",
terraform_data.ansible_playbook (local-exec):         "     Memory: 971.6M",
terraform_data.ansible_playbook (local-exec):         "        CPU: 4min 8.511s",
terraform_data.ansible_playbook (local-exec):         "     CGroup: /system.slice/puppetserver.service",
terraform_data.ansible_playbook (local-exec):         "             └─25102 /usr/bin/java --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED -Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger -Dlogappender=F1 -XX:+CrashOnOutOfMemoryError -XX:ErrorFile=/var/log/puppetlabs/puppetserver/puppetserver_err_pid%p.log -cp \"/opt/puppetlabs/server/apps/puppetserver/puppet-server-release.jar:/opt/puppetlabs/server/data/puppetserver/jars/*\" clojure.main -m puppetlabs.trapperkeeper.main --config /etc/puppetlabs/puppetserver/conf.d --bootstrap-config /etc/puppetlabs/puppetserver/services.d/,/opt/puppetlabs/server/apps/puppetserver/config/services.d/ --restart-file /opt/puppetlabs/server/data/puppetserver/restartcounter",
terraform_data.ansible_playbook (local-exec):         "",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:49:49 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:49:54 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:49:54 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:49:55 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:50:01 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:50:14 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:50:15 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:50:18 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:50:18 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
terraform_data.ansible_playbook (local-exec):         "Jul 09 11:51:20 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether."
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }

terraform_data.ansible_playbook (local-exec): TASK [Wait 20 seconds for Agents and PuppetDB to request a certificate] ********
terraform_data.ansible_playbook: Still creating... [20s elapsed]
terraform_data.ansible_playbook: Still creating... [30s elapsed]
terraform_data.ansible_playbook (local-exec): Pausing for 20 seconds
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Sign CA from Puppet Agents] **********************************************
terraform_data.ansible_playbook: Still creating... [40s elapsed]
terraform_data.ansible_playbook (local-exec): fatal: [10.100.65.79]: FAILED! => {"changed": true, "cmd": ["/opt/puppetlabs/bin/puppetserver", "ca", "sign", "--all"], "delta": "0:00:00.376882", "end": "2024-07-10 00:59:44.909978", "msg": "non-zero return code", "rc": 24, "start": "2024-07-10 00:59:44.533096", "stderr": "Error:\n    No waiting certificate requests to sign", "stderr_lines": ["Error:", "    No waiting certificate requests to sign"], "stdout": "", "stdout_lines": []}
terraform_data.ansible_playbook (local-exec): ...ignoring

terraform_data.ansible_playbook (local-exec): TASK [Install PuppetDB module in PuppetServer] *********************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Show result of module installation] **************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Module puppetlabs-puppetdb 8.1.0 is already installed.\u001b[0m"
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }

terraform_data.ansible_playbook (local-exec): TASK [Create site.pp] **********************************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Edit site.pp] ************************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): TASK [Apply Site.pp in Puppet Server] ******************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.65.79]

terraform_data.ansible_playbook (local-exec): PLAY [puppetagents puppetdb] ***************************************************

terraform_data.ansible_playbook (local-exec): TASK [Test communication between Agents , PuppetDB and PuppetServer] ***********
terraform_data.ansible_playbook: Still creating... [50s elapsed]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.13]
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Show test result] ********************************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.13] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Loading facts\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Caching catalog for tf-puppetagent-0.openstacklocal\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Applying configuration version '1720573199'\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Applied catalog in 0.01 seconds\u001b[0m"
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Loading facts\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Caching catalog for tf-puppetdb.openstacklocal\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[0;32mInfo: Applying configuration version '1720573200'\u001b[0m",
terraform_data.ansible_playbook (local-exec):         "\u001b[mNotice: Applied catalog in 1.26 seconds\u001b[0m"
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }

terraform_data.ansible_playbook (local-exec): PLAY [puppetdb] ****************************************************************

terraform_data.ansible_playbook (local-exec): TASK [Verify opened port for PuppetDB] *****************************************
terraform_data.ansible_playbook (local-exec): changed: [10.100.67.48]

terraform_data.ansible_playbook (local-exec): TASK [Show opened port tcp/8081] ***********************************************
terraform_data.ansible_playbook (local-exec): ok: [10.100.67.48] => {
terraform_data.ansible_playbook (local-exec):     "result.stdout_lines": [
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      4096           127.0.0.53%lo:53                  0.0.0.0:*     users:((\"systemd-resolve\",pid=541,fd=14))                                         ",
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      128                  0.0.0.0:22                  0.0.0.0:*     users:((\"sshd\",pid=832,fd=3))                                                     ",
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      244                  0.0.0.0:5432                0.0.0.0:*     users:((\"postgres\",pid=23023,fd=5))                                               ",
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      128                     [::]:22                     [::]:*     users:((\"sshd\",pid=832,fd=4))                                                     ",
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      50        [::ffff:127.0.0.1]:8080                      *:*     users:((\"java\",pid=25058,fd=10))                                                  ",
terraform_data.ansible_playbook (local-exec):         "LISTEN    0      50                         *:8081                      *:*     users:((\"java\",pid=25058,fd=11))                                                  "
terraform_data.ansible_playbook (local-exec):     ]
terraform_data.ansible_playbook (local-exec): }

terraform_data.ansible_playbook (local-exec): PLAY RECAP *********************************************************************
terraform_data.ansible_playbook (local-exec): 10.100.65.79               : ok=17   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
terraform_data.ansible_playbook (local-exec): 10.100.67.13               : ok=12   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
terraform_data.ansible_playbook (local-exec): 10.100.67.48               : ok=14   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

terraform_data.ansible_playbook: Still creating... [1m0s elapsed]
terraform_data.ansible_playbook: Creation complete after 1m1s [id=0d8205ad-da2e-1035-06f0-54a12aae98b9]

Apply complete! Resources: 1 added, 3 changed, 1 destroyed.
```
