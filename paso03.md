# CHALLENGE 03  PASO 3: DESPLEGAR INFRAESTRUCTURA


Usamos el comando "terraform init" para instalar los plugin necesarios para la comunicación de Terraform hacia Openstack

```
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding terraform-provider-openstack/openstack versions matching "1.53.0"...
- Installing terraform-provider-openstack/openstack v1.53.0...
- Installed terraform-provider-openstack/openstack v1.53.0 (self-signed, key ID 4F80527A391BEFD2)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!
```

Luego usamos los siguientes comandos para validar  los scripts antes de aplicarlos

```
challenger-16@challenge-3-pivote:~/terraform-dir$ terraform fmt

challenger-16@challenge-3-pivote:~/terraform-dir$ terraform validate
Success! The configuration is valid.

challenger-16@challenge-3-pivote:~/terraform-dir$ terraform plan
### output omitido por brevedad
Plan: 2 to add, 0 to change, 0 to destroy.
```

Ahora aplicamos los scripts. Recordemos que en el paso 02 creamos un archivo "variables.tf" donde tenemos la variable puppetagentcount con el valor 1 por defecto.  Al hacer el "apply" podemos modificar al vuelo el contenido de la variable. En este ejemplo hacemos que puppetagentcount sea igual a 2.

Cuando se hace el "apply" nos sale la pregunta "Do you want to perform these actions?" Es obligatorio escribir "yes" para continuar


```
challenger-16@challenge-3-pivote:~/terraform-dir$  terraform apply -var "puppetagentcount=2"
Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


### output omitido por brevedad

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

```

Ahora podemos usar los comandos de Openstack para validar que las instancias y demás recursos fueron creados exitosamente

```
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack server list
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| ID                                   | Name             | Status | Networks            | Image                    | Flavor   |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| 9f1b07fe-8ac7-4ab6-8dd4-845be251a1f3 | tf_puppetagent_1 | ACTIVE | PUBLIC=10.100.66.16 | N/A (booted from volume) | m1.small |
| 3f084247-ca12-4e5d-8c98-eb84299ec809 | tf_puppetserver  | ACTIVE | PUBLIC=10.100.65.79 | N/A (booted from volume) | m1.small |
| 5a57ac29-0757-4fc1-8816-2c4d94cc7b04 | tf_puppetagent_0 | ACTIVE | PUBLIC=10.100.67.13 | N/A (booted from volume) | m1.small |
| 8f260cc7-26b4-4bd6-9151-ba1d3712ed70 | tf_puppetdb      | ACTIVE | PUBLIC=10.100.67.48 | N/A (booted from volume) | m1.small |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
challenger-16@challenge-3-pivote:~/terraform-dir$
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack volume list
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+
| ID                                   | Name                  | Status | Size | Attached to                               |
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+
| 8606f116-0dd8-40dd-bda1-5588e68e9abb | tf_puppetagents_vol_1 | in-use |   20 | Attached to tf_puppetagent_1 on /dev/vda  |
| d55b6118-f5d7-4e4b-a222-c45c84a7323a | tf_puppetagents_vol_0 | in-use |   20 | Attached to tf_puppetagent_0 on /dev/vda  |
| 6cb520de-5b00-4af7-b40d-544222f60cce | tf_puppetdb_vol       | in-use |   20 | Attached to tf_puppetdb on /dev/vda       |
| 25e39d6f-f2af-433f-b98a-0f0c6eb37c84 | tf_puppeserver_vol    | in-use |   20 | Attached to tf_puppetserver on /dev/vda   |
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack security group list
+--------------------------------------+--------------------+----------------------------+----------------------------------+------+
| ID                                   | Name               | Description                | Project                          | Tags |
+--------------------------------------+--------------------+----------------------------+----------------------------------+------+
| 12073ba4-a5b3-4872-a3ba-4a9a7d236f1c | tf_puppetdb_sg     | terraform puppet db sg     | 46c6e85c6f52456ba03c2d3ecab15e13 | []   |
| 54fc883a-fde8-452d-bcf3-524697e937fb | tf_puppetserver_sg | terraform puppet server sg | 46c6e85c6f52456ba03c2d3ecab15e13 | []   |
| 87155be4-2e5a-448e-ac95-91105c1f5d7d | tf_puppetagents_sg | terraform puppet agents sg | 46c6e85c6f52456ba03c2d3ecab15e13 | []   |
| b83d1cef-f993-4ab4-90ca-b735063aacc1 | default            | Default security group     | 46c6e85c6f52456ba03c2d3ecab15e13 | []   |
+--------------------------------------+--------------------+----------------------------+----------------------------------+------+

```

Si ahora hacemos puppetagentcount=1 vamos a ver que las instancias de PuppetAgent se reducen a sólo 1 instancia.


```
challenger-16@challenge-3-pivote:~/terraform-dir$  terraform apply -var "puppetagentcount=1"

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

### output omitido por brevedad

Apply complete! Resources: 0 added, 0 changed, 2 destroyed.
```

Volvemos a validar con los comandos de Openstack y vemos que ahora hay sólo 1 instancia de Puppet Agent y 1 sólo volume dedicado a Puppet Agent

```
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack server list
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| ID                                   | Name             | Status | Networks            | Image                    | Flavor   |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
| 3f084247-ca12-4e5d-8c98-eb84299ec809 | tf_puppetserver  | ACTIVE | PUBLIC=10.100.65.79 | N/A (booted from volume) | m1.small |
| 5a57ac29-0757-4fc1-8816-2c4d94cc7b04 | tf_puppetagent_0 | ACTIVE | PUBLIC=10.100.67.13 | N/A (booted from volume) | m1.small |
| 8f260cc7-26b4-4bd6-9151-ba1d3712ed70 | tf_puppetdb      | ACTIVE | PUBLIC=10.100.67.48 | N/A (booted from volume) | m1.small |
+--------------------------------------+------------------+--------+---------------------+--------------------------+----------+
challenger-16@challenge-3-pivote:~/terraform-dir$
challenger-16@challenge-3-pivote:~/terraform-dir$ openstack volume list
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+
| ID                                   | Name                  | Status | Size | Attached to                               |
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+
| d55b6118-f5d7-4e4b-a222-c45c84a7323a | tf_puppetagents_vol_0 | in-use |   20 | Attached to tf_puppetagent_0 on /dev/vda  |
| 6cb520de-5b00-4af7-b40d-544222f60cce | tf_puppetdb_vol       | in-use |   20 | Attached to tf_puppetdb on /dev/vda       |
| 25e39d6f-f2af-433f-b98a-0f0c6eb37c84 | tf_puppeserver_vol    | in-use |   20 | Attached to tf_puppetserver on /dev/vda   |
+--------------------------------------+-----------------------+--------+------+-------------------------------------------+

```