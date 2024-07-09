# CHALLENGE 03  PASO 05: CONFIGURAR E INSTALAR PUPPET

En este paso ejecutamos el playbook que creamos en el paso anterior.
En el output de abajo podemos ver el status de los servicios puppetserver y puppet que confirman que los servicios funcionan correctamente.
Además PuppetServer firmó los certificado de PuppetAgent y PuppetDB.
Los últimos tasks indican que PuppetAgent y PuppetDB lograron comunicación SSL exitosa con PuppetServer. 
Por otro lado PuppetDB abrió puerto tcp/8081 y además tcp/5432 para uso de Postgres

```
challenger-16@challenge-3-pivote:~/ansible-dir$ ansible-playbook -i myinventory.ini myplay.yml

PLAY [all] *******************************************************************************************************************************************************

TASK [Edit /etc/hosts file] **************************************************************************************************************************************
ok: [puppetserver]
ok: [puppetdb]
ok: [puppetagent_0]

TASK [Download Puppet.deb] ***************************************************************************************************************************************
ok: [puppetdb]
ok: [puppetserver]
ok: [puppetagent_0]

TASK [Install Puppet repository] *********************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetserver]
ok: [puppetdb]

TASK [Apt update] ************************************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]
changed: [puppetserver]

PLAY [puppetagents puppetdb] *************************************************************************************************************************************

TASK [Install Puppet Agent] **************************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Avoid kernel warning popups] *******************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Avoid services need to be restarted warning popups] ********************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Start and enable services] *********************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Check service status] **************************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Show service status] ***************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "● puppet.service - Puppet agent",
        "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 08:06:55 UTC; 13h ago",
        "       Docs: man:puppet-agent(8)",
        "   Main PID: 8854 (puppet)",
        "      Tasks: 2 (limit: 2324)",
        "     Memory: 77.0M",
        "        CPU: 2min 1.367s",
        "     CGroup: /system.slice/puppet.service",
        "             └─8854 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
        "",
        "Jul 09 20:07:02 tf-puppetagent-0 puppet-agent[29226]: Applied catalog in 0.01 seconds",
        "Jul 09 20:37:02 tf-puppetagent-0 puppet-agent[29351]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 20:37:02 tf-puppetagent-0 puppet-agent[29351]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 20:37:02 tf-puppetagent-0 puppet-agent[29351]: Applied catalog in 0.01 seconds",
        "Jul 09 21:07:03 tf-puppetagent-0 puppet-agent[29472]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 21:07:03 tf-puppetagent-0 puppet-agent[29472]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 21:07:03 tf-puppetagent-0 puppet-agent[29472]: Applied catalog in 0.01 seconds",
        "Jul 09 21:37:02 tf-puppetagent-0 puppet-agent[30080]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 21:37:02 tf-puppetagent-0 puppet-agent[30080]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 21:37:02 tf-puppetagent-0 puppet-agent[30080]: Applied catalog in 0.01 seconds"
    ]
}
ok: [puppetdb] => {
    "result.stdout_lines": [
        "● puppet.service - Puppet agent",
        "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 08:39:50 UTC; 13h ago",
        "       Docs: man:puppet-agent(8)",
        "   Main PID: 10355 (puppet)",
        "      Tasks: 2 (limit: 2324)",
        "     Memory: 77.2M",
        "        CPU: 2min 52.510s",
        "     CGroup: /system.slice/puppet.service",
        "             └─10355 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
        "",
        "Jul 09 20:10:00 tf-puppetdb puppet-agent[39733]: Applied catalog in 1.27 seconds",
        "Jul 09 20:39:56 tf-puppetdb puppet-agent[40134]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 20:39:58 tf-puppetdb puppet-agent[40134]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 20:40:00 tf-puppetdb puppet-agent[40134]: Applied catalog in 1.27 seconds",
        "Jul 09 21:09:57 tf-puppetdb puppet-agent[40466]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 21:09:59 tf-puppetdb puppet-agent[40466]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 21:10:00 tf-puppetdb puppet-agent[40466]: Applied catalog in 1.27 seconds",
        "Jul 09 21:39:56 tf-puppetdb puppet-agent[40885]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 21:39:59 tf-puppetdb puppet-agent[40885]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 21:40:00 tf-puppetdb puppet-agent[40885]: Applied catalog in 1.28 seconds"
    ]
}

PLAY [puppetserver] **********************************************************************************************************************************************

TASK [Install Puppet Server] *************************************************************************************************************************************
ok: [puppetserver]

TASK [Change Java options Xms to reduce memory usage] ************************************************************************************************************
changed: [puppetserver]

TASK [Change Java Xmx to reduce memory usage] ********************************************************************************************************************
changed: [puppetserver]

TASK [Start and enable services] *********************************************************************************************************************************
ok: [puppetserver]

TASK [Check service status] **************************************************************************************************************************************
changed: [puppetserver]

TASK [Show service status] ***************************************************************************************************************************************
ok: [puppetserver] => {
    "result.stdout_lines": [
        "● puppetserver.service - puppetserver Service",
        "     Loaded: loaded (/lib/systemd/system/puppetserver.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 09:24:10 UTC; 12h ago",
        "   Main PID: 25102 (java)",
        "      Tasks: 54 (limit: 4915)",
        "     Memory: 965.9M",
        "        CPU: 3min 43.562s",
        "     CGroup: /system.slice/puppetserver.service",
        "             └─25102 /usr/bin/java --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED -Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger -Dlogappender=F1 -XX:+CrashOnOutOfMemoryError -XX:ErrorFile=/var/log/puppetlabs/puppetserver/puppetserver_err_pid%p.log -cp \"/opt/puppetlabs/server/apps/puppetserver/puppet-server-release.jar:/opt/puppetlabs/server/data/puppetserver/jars/*\" clojure.main -m puppetlabs.trapperkeeper.main --config /etc/puppetlabs/puppetserver/conf.d --bootstrap-config /etc/puppetlabs/puppetserver/services.d/,/opt/puppetlabs/server/apps/puppetserver/config/services.d/ --restart-file /opt/puppetlabs/server/data/puppetserver/restartcounter",
        "",
        "Jul 09 11:49:49 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:49:54 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:49:54 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:49:55 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:50:01 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:50:14 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:50:15 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:50:18 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:50:18 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether.",
        "Jul 09 11:51:20 tf-puppetserver systemd[1]: /lib/systemd/system/puppetserver.service:45: Standard output type syslog is obsolete, automatically updating to journal. Please update your unit file, and consider removing the setting altogether."
    ]
}

TASK [Wait 20 seconds for Agents and PuppetDB to request a certificate] ******************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [puppetserver]

TASK [Sign CA from Puppet Agents] ********************************************************************************************************************************
fatal: [puppetserver]: FAILED! => {"changed": true, "cmd": ["/opt/puppetlabs/bin/puppetserver", "ca", "sign", "--all"], "delta": "0:00:00.330376", "end": "2024-07-09 21:57:08.560737", "msg": "non-zero return code", "rc": 24, "start": "2024-07-09 21:57:08.230361", "stderr": "Error:\n    No waiting certificate requests to sign", "stderr_lines": ["Error:", "    No waiting certificate requests to sign"], "stdout": "", "stdout_lines": []}
...ignoring

TASK [Install PuppetDB module in PuppetServer] *******************************************************************************************************************
changed: [puppetserver]

TASK [Show result of module installation] ************************************************************************************************************************
ok: [puppetserver] => {
    "result.stdout_lines": [
        "\u001b[mNotice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...\u001b[0m",
        "\u001b[mNotice: Module puppetlabs-puppetdb 8.1.0 is already installed.\u001b[0m"
    ]
}

TASK [Create site.pp] ********************************************************************************************************************************************
changed: [puppetserver]

TASK [Edit site.pp] **********************************************************************************************************************************************
ok: [puppetserver]

TASK [Apply Site.pp in Puppet Server] ****************************************************************************************************************************
changed: [puppetserver]

PLAY [puppetagents puppetdb] *************************************************************************************************************************************

TASK [Test communication between Agents , PuppetDB and PuppetServer] *********************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Show test result] ******************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
        "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
        "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
        "\u001b[0;32mInfo: Loading facts\u001b[0m",
        "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
        "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Caching catalog for tf-puppetagent-0.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Applying configuration version '1720562243'\u001b[0m",
        "\u001b[mNotice: Applied catalog in 0.01 seconds\u001b[0m"
    ]
}
ok: [puppetdb] => {
    "result.stdout_lines": [
        "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
        "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
        "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
        "\u001b[0;32mInfo: Loading facts\u001b[0m",
        "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
        "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Caching catalog for tf-puppetdb.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Applying configuration version '1720562243'\u001b[0m",
        "\u001b[mNotice: Applied catalog in 1.27 seconds\u001b[0m"
    ]
}

PLAY [puppetdb] **************************************************************************************************************************************************

TASK [Verify opened port for PuppetDB] ***************************************************************************************************************************
changed: [puppetdb]

TASK [Show opened port tcp/8081] *********************************************************************************************************************************
ok: [puppetdb] => {
    "result.stdout_lines": [
        "LISTEN    0      4096           127.0.0.53%lo:53                  0.0.0.0:*     users:((\"systemd-resolve\",pid=541,fd=14))                                         ",
        "LISTEN    0      128                  0.0.0.0:22                  0.0.0.0:*     users:((\"sshd\",pid=832,fd=3))                                                     ",
        "LISTEN    0      244                  0.0.0.0:5432                0.0.0.0:*     users:((\"postgres\",pid=23023,fd=5))                                               ",
        "LISTEN    0      128                     [::]:22                     [::]:*     users:((\"sshd\",pid=832,fd=4))                                                     ",
        "LISTEN    0      50        [::ffff:127.0.0.1]:8080                      *:*     users:((\"java\",pid=25058,fd=10))                                                  ",
        "LISTEN    0      50                         *:8081                      *:*     users:((\"java\",pid=25058,fd=11))                                                  "
    ]
}

PLAY RECAP *******************************************************************************************************************************************************
puppetagent_0              : ok=12   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetdb                   : ok=14   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetserver               : ok=17   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```
