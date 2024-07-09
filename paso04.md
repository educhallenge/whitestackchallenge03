# CHALLENGE 03  PASO 4: CREAR PLAYBOOKS DE ANSIBLE

## VERIFICAR QUE ANSIBLE YA ESTA INSTALADO


En nuestro Ubuntu remoto verificamos que Ansible ya está instalado
```
challenger-16@challenge-3-pivote:~$ ansible --version
ansible [core 2.16.7]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/challenger-16/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/challenger-16/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Apr 10 2024, 05:33:47) [GCC 13.2.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

Creamos una carpeta que servirá para almacenar el inventario y playbooks de Ansible

```
challenger-16@challenge-3-pivote:~$ mkdir ansible-dir
challenger-16@challenge-3-pivote:~$ cd ansible-dir/

```

## CREAR INVENTARIO


Creamos el archivo "myinventory.ini" que servirá como inventario de los playbooks que vamos a crear.

```
challenger-16@challenge-3-pivote:~/ansible-dir$ echo '
puppetserver ansible_host=10.100.65.79
puppetdb ansible_host=10.100.67.48

[puppetagents]
puppetagent_0 ansible_host=10.100.67.13

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/challenger-16/mykey.PEM
' > myinventory.ini
```

Verificamos la conectividad SSH desde nuestro Ubuntu remoto hacia cada host del inventario

```
challenger-16@challenge-3-pivote:~/ansible-dir$ ansible all --list-hosts -i myinventory.ini
  hosts (3):
    puppetserver
    puppetdb
    puppetagent_0

challenger-16@challenge-3-pivote:~/ansible-dir$ ansible all -m ping -i myinventory.ini
puppetdb | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
puppetserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
puppetagent_0 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```


## CREAR PLAYBOOK

Creamos el playbook para desplegar PuppetServer, PuppetAgent.


```
---
- hosts: all
  gather_facts: no
  tasks:
   - name: Edit /etc/hosts file
     become: yes
     ansible.builtin.blockinfile:
       path: /etc/hosts
       block: |
         10.100.65.79  puppet puppetserver tf-puppetserver.openstacklocal
         10.100.67.48 tf-puppetdb.openstacklocal
       insertafter: "EOF"


   - name: Download Puppet.deb
#     become: yes
     get_url:
       url: https://apt.puppet.com/puppet8-release-jammy.deb
       dest: /home/ubuntu

   - name: Install Puppet repository
     become: yes
     apt:
        deb: /home/ubuntu/puppet8-release-jammy.deb

   - name: Apt update
     become: yes
     apt:
        update_cache: yes

- hosts: puppetserver
  gather_facts: no
  tasks:
   - name: Install Puppet Server
     become: yes
     apt:
       name: puppetserver
       state: present

   - name: Change Java options Xms to reduce memory usage
     action: shell sudo sed -i 's/Xms2g/Xms512m/' /etc/default/puppetserver

   - name: Change Java Xmx to reduce memory usage
     action: shell sudo sed -i 's/Xmx2g/Xmx512m/' /etc/default/puppetserver

   - name: Start and enable services
     become: yes
     systemd:
      name: puppetserver
      state: started
      enabled: yes

   - name: Check service status
     become: yes
     command: systemctl status puppetserver
     register: result

   - name: Show service status
     debug:
       var: result.stdout_lines

- hosts: [puppetagents puppetdb]
  gather_facts: no
  tasks:
   - name: Install Puppet Agent
     become: yes
     apt:
       name: puppet-agent
       state: present

   - name: Avoid kernel warning popups
     action: shell sudo sed -i "s/#\$nrconf{kernelhints} = -1;/\$nrconf{kernelhints} = -1;/g" /etc/needrestart/needrestart.conf

   - name: Avoid services need to be restarted warning popups
     action: shell sudo sed -i "/#\$nrconf{restart} = 'i';/s/.*/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf

   - name: Start and enable services
     become: yes
     systemd:
      name: puppet
      state: started
      enabled: yes

   - name: Check service status
     become: yes
     command: systemctl status puppet
     register: result

   - name: Show service status
     debug:
       var: result.stdout_lines

- hosts: puppetserver
  gather_facts: no
  tasks:
   - name: Sign CA from Puppet Agents
     become: yes
     command: /opt/puppetlabs/bin/puppetserver ca sign --all
     ignore_errors: true

   - name: Install PuppetDB module in PuppetServer
     become: yes
     command: sudo /opt/puppetlabs/bin/puppet  module install puppetlabs-puppetdb --version 8.1.0
     register: result

   - name: Show result of module installation
     debug:
       var: result.stdout_lines

   - name: Create site.pp
     become: yes
     file:
       path: "/etc/puppetlabs/code/environments/production/manifests/site.pp"
       state: touch

   - name: Edit site.pp
     become: yes
     ansible.builtin.blockinfile:
       path: /etc/puppetlabs/code/environments/production/manifests/site.pp
       block: |
        node 'default' {}
        node 'tf-puppetserver.openstacklocal' {
          class { 'puppetdb::master::config':
            puppetdb_server => 'tf-puppetdb.openstacklocal',
          }
        }

        node 'tf-puppetdb.openstacklocal' {
          class { 'puppetdb':
            database_host => 'tf-puppetdb.openstacklocal',
            database_listen_address => '0.0.0.0'
          }
        }
       insertafter: "EOF"

   - name: Apply Site.pp in Puppet Server
     become: yes
     command: sudo /opt/puppetlabs/bin/puppet apply  /etc/puppetlabs/code/environments/production/manifests/site.pp


- hosts: [puppetagents puppetdb]
  gather_facts: no
  tasks:
   - name: Test communication between Agents , PuppetDB and PuppetServer
     become: yes
     command: sudo /opt/puppetlabs/bin/puppet agent --test
     register: result

   - name: Show test result
     debug:
       var: result.stdout_lines

- hosts: puppetdb
  gather_facts: no
  tasks:
   - name: Verify opened port for PuppetDB
     become: yes
     shell: sudo ss -aontp | grep LISTEN
     register: result

   - name: Show opened port tcp/8081
     debug:
       var: result.stdout_lines
' > myplay.yml
```
