# CHALLENGE 03: PASO 6

Crear archivo de inventario que luego recibirá dinámicante la información de los hostnames y direcciones IP de las instancias que Terraform creará.




Crear archivo "ansible.tf"
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
