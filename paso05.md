# CHALLENGE 03  PASO 05: CONFIGURAR E INSTALAR PUPPET

En este paso ejecutamos el playbook que creamos en el paso anterior.
En el output de abajo podemos ver el status de los servicios puppetserver y puppet que confirman que los servicios funcionan correctamente.
Además PuppetServer firmó los certificado de PuppetAgent y PuppetDB.
Los últimos tasks indican que PuppetAgent logró comunicación SSL exitosa con PuppetServer. Por otro lado PuppetDB además de lograr comunicación SSL exitosa con PuppetServer también realizó la instalación de PostgreSQL . 
El output del último task indica que PuppetDB abrió el puerto 8081 el cual usa para la base de datos.

```
challenger-16@challenge-3-pivote:~/ansible-dir$ ansible-playbook -i myinventory.ini myplay.yml

PLAY [all] ******************************************************************************************************************************************************

TASK [Edit /etc/hosts file] *************************************************************************************************************************************
ok: [puppetdb]
ok: [puppetagent_0]
ok: [puppetserver]

TASK [Download Puppet.deb] **************************************************************************************************************************************
ok: [puppetdb]
ok: [puppetagent_0]
ok: [puppetserver]

TASK [Install Puppet repository] ********************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetdb]
ok: [puppetserver]

TASK [Apt update] ***********************************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetserver]
changed: [puppetdb]

PLAY [puppetserver] *********************************************************************************************************************************************

TASK [Install Puppet Server] ************************************************************************************************************************************
ok: [puppetserver]

TASK [Change Java options Xms to reduce memory usage] ***********************************************************************************************************
changed: [puppetserver]

TASK [Change Java Xmx to reduce memory usage] *******************************************************************************************************************
changed: [puppetserver]

TASK [Start and enable services] ********************************************************************************************************************************
ok: [puppetserver]

TASK [Check service status] *************************************************************************************************************************************
changed: [puppetserver]

TASK [Show service status] **************************************************************************************************************************************
ok: [puppetserver] => {
    "result.stdout_lines": [
        "● puppetserver.service - puppetserver Service",
        "     Loaded: loaded (/lib/systemd/system/puppetserver.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 09:24:10 UTC; 2min 45s ago",
        "    Process: 25053 ExecStart=/opt/puppetlabs/server/apps/puppetserver/bin/puppetserver start (code=exited, status=0/SUCCESS)",
        "   Main PID: 25102 (java)",
        "      Tasks: 59 (limit: 4915)",
        "     Memory: 865.4M",
        "        CPU: 51.654s",
        "     CGroup: /system.slice/puppetserver.service",
        "             ├─25102 /usr/bin/java --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED -Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger -Dlogappender=F1 -XX:+CrashOnOutOfMemoryError -XX:ErrorFile=/var/log/puppetlabs/puppetserver/puppetserver_err_pid%p.log -cp \"/opt/puppetlabs/server/apps/puppetserver/puppet-server-release.jar:/opt/puppetlabs/server/data/puppetserver/jars/*\" clojure.main -m puppetlabs.trapperkeeper.main --config /etc/puppetlabs/puppetserver/conf.d --bootstrap-config /etc/puppetlabs/puppetserver/services.d/,/opt/puppetlabs/server/apps/puppetserver/config/services.d/ --restart-file /opt/puppetlabs/server/data/puppetserver/restartcounter",
        "             └─25216 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/server/data/puppetserver/dropsonde/bin/dropsonde submit",
        "",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,737 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@3924a8b4 - The date pattern is 'yyyy-MM-dd' from file name pattern '/var/log/puppetlabs/puppetserver/puppetserver-access-%d{yyyy-MM-dd}.%i.log.gz'.",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,737 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@3924a8b4 - Roll-over at midnight.",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,737 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@3924a8b4 - Setting initial period to 2024-07-09T09:23:34.743Z",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,737 |-INFO in ch.qos.logback.core.model.processor.ImplicitModelHandler - Assuming default type [ch.qos.logback.access.PatternLayoutEncoder] for [encoder] property",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,742 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[FILE] - Active log file name: /var/log/puppetlabs/puppetserver/puppetserver-access.log",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,742 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[FILE] - File property is set to [/var/log/puppetlabs/puppetserver/puppetserver-access.log]",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,742 |-INFO in ch.qos.logback.core.model.processor.AppenderRefModelHandler - Attaching appender named [FILE] to null",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,742 |-INFO in ch.qos.logback.core.model.processor.DefaultProcessor@3bf251 - End of configuration.",
        "Jul 09 09:24:09 tf-puppetserver puppetserver[25102]: 09:24:09,742 |-INFO in ch.qos.logback.access.joran.JoranConfigurator@d385597 - Registering current configuration as safe fallback point",
        "Jul 09 09:24:10 tf-puppetserver systemd[1]: Started puppetserver Service."
    ]
}

PLAY [puppetagents puppetdb] ************************************************************************************************************************************

TASK [Install Puppet Agent] *************************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Avoid kernel warning popups] ******************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Avoid services need to be restarted warning popups] *******************************************************************************************************
changed: [puppetdb]
changed: [puppetagent_0]

TASK [Start and enable services] ********************************************************************************************************************************
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Check service status] *************************************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Show service status] **************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "● puppet.service - Puppet agent",
        "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 08:06:55 UTC; 1h 20min ago",
        "       Docs: man:puppet-agent(8)",
        "   Main PID: 8854 (puppet)",
        "      Tasks: 2 (limit: 2324)",
        "     Memory: 75.9M",
        "        CPU: 8.446s",
        "     CGroup: /system.slice/puppet.service",
        "             └─8854 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
        "",
        "Jul 09 08:13:30 tf-puppetagent-0 puppet-agent[9881]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 08:13:30 tf-puppetagent-0 puppet-agent[9881]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 08:13:30 tf-puppetagent-0 puppet-agent[9881]: Applied catalog in 0.01 seconds",
        "Jul 09 08:36:58 tf-puppetagent-0 puppet-agent[9930]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 08:36:58 tf-puppetagent-0 puppet-agent[9930]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 08:36:58 tf-puppetagent-0 puppet-agent[9930]: Applied catalog in 0.01 seconds",
        "Jul 09 09:07:02 tf-puppetagent-0 puppet-agent[16911]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 09:07:02 tf-puppetagent-0 puppet-agent[16911]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 09:07:02 tf-puppetagent-0 puppet-agent[16911]: Could not retrieve catalog from remote server: Error 500 on SERVER: Server Error: Could not find node statement with name 'default' or 'tf-puppetagent-0.openstacklocal' on node tf-puppetagent-0.openstacklocal",
        "Jul 09 09:07:02 tf-puppetagent-0 puppet-agent[16911]: Applied catalog in 0.01 seconds"
    ]
}
ok: [puppetdb] => {
    "result.stdout_lines": [
        "● puppet.service - Puppet agent",
        "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 08:39:50 UTC; 47min ago",
        "       Docs: man:puppet-agent(8)",
        "   Main PID: 10355 (puppet)",
        "      Tasks: 2 (limit: 2324)",
        "     Memory: 75.1M",
        "        CPU: 22.197s",
        "     CGroup: /system.slice/puppet.service",
        "             └─10355 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
        "",
        "Jul 09 08:39:50 tf-puppetdb systemd[1]: Started Puppet agent.",
        "Jul 09 08:39:51 tf-puppetdb puppet-agent[10355]: Starting Puppet client version 8.7.0",
        "Jul 09 08:42:37 tf-puppetdb puppet[10355]: Couldn't fetch certificate from CA server; you might still need to sign this agent's certificate (tf-puppetdb.openstacklocal).",
        "Jul 09 08:42:38 tf-puppetdb puppet-agent[11322]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 08:42:39 tf-puppetdb puppet-agent[11322]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 08:42:39 tf-puppetdb puppet-agent[11322]: Applied catalog in 0.01 seconds",
        "Jul 09 09:09:56 tf-puppetdb puppet-agent[17542]: Requesting catalog from puppet:8140 (10.100.65.79)",
        "Jul 09 09:09:56 tf-puppetdb puppet-agent[17542]: Catalog compiled by tf-puppetserver.openstacklocal",
        "Jul 09 09:09:56 tf-puppetdb puppet-agent[17542]: Could not retrieve catalog from remote server: Error 500 on SERVER: Server Error: Could not find node statement with name 'default' or 'tf-puppetdb.openstacklocal' on node tf-puppetdb.openstacklocal",
        "Jul 09 09:09:56 tf-puppetdb puppet-agent[17542]: Applied catalog in 0.01 seconds"
    ]
}

PLAY [puppetserver] *********************************************************************************************************************************************

TASK [Sign CA from Puppet Agents] *******************************************************************************************************************************
fatal: [puppetserver]: FAILED! => {"changed": true, "cmd": ["/opt/puppetlabs/bin/puppetserver", "ca", "sign", "--all"], "delta": "0:00:00.658183", "end": "2024-07-09 09:27:00.433979", "msg": "non-zero return code", "rc": 24, "start": "2024-07-09 09:26:59.775796", "stderr": "Error:\n    No waiting certificate requests to sign", "stderr_lines": ["Error:", "    No waiting certificate requests to sign"], "stdout": "", "stdout_lines": []}
...ignoring

TASK [Install PuppetDB module in PuppetServer] ******************************************************************************************************************
changed: [puppetserver]

TASK [Show result of module installation] ***********************************************************************************************************************
ok: [puppetserver] => {
    "result.stdout_lines": [
        "\u001b[mNotice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...\u001b[0m",
        "\u001b[mNotice: Module puppetlabs-puppetdb 8.1.0 is already installed.\u001b[0m"
    ]
}

TASK [Create site.pp] *******************************************************************************************************************************************
changed: [puppetserver]

TASK [Edit site.pp] *********************************************************************************************************************************************
ok: [puppetserver]

TASK [Apply Site.pp in Puppet Server] ***************************************************************************************************************************
changed: [puppetserver]

PLAY [puppetagents puppetdb] ************************************************************************************************************************************

TASK [Test communication between Agents and Server] *************************************************************************************************************
changed: [puppetagent_0]
changed: [puppetdb]

TASK [Show test result] *****************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
        "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
        "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
        "\u001b[0;32mInfo: Loading facts\u001b[0m",
        "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
        "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Caching catalog for tf-puppetagent-0.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Applying configuration version '1720517235'\u001b[0m",
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
        "\u001b[0;32mInfo: Applying configuration version '1720517236'\u001b[0m",
        "\u001b[mNotice: Applied catalog in 1.27 seconds\u001b[0m"
    ]
}

PLAY RECAP ******************************************************************************************************************************************************
puppetagent_0              : ok=12   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetdb                   : ok=12   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetserver               : ok=16   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```
