# CHALLENGE 03  PASO 05: CONFIGURAR E INSTALAR PUPPET

En este paso ejecutamos el playbook que creamos en el paso anterior.
En el output de abajo podemos ver el status de los servicios puppetserver y puppet que confirman que los servicios funcionan correctamente.
Además el output de los últimos tasks nos indican que PuppetServer firmó el certificado de PuppetAgent y con ello PuppetAgent logró comunicación SSL con PuppetServer.

```
challenger-16@challenge-3-pivote:~/ansible-dir$ ansible-playbook -i myinventory.ini myplay.yml

PLAY [all] ****************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [puppetdb]
ok: [puppetagent_0]
ok: [puppetserver]

TASK [Edit /etc/hosts file] ***********************************************************************************************************************************
ok: [puppetdb]
ok: [puppetserver]
ok: [puppetagent_0]

TASK [Download Puppet.deb] ************************************************************************************************************************************
ok: [puppetserver]
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Install Puppet repository] ******************************************************************************************************************************
ok: [puppetserver]
ok: [puppetagent_0]
ok: [puppetdb]

TASK [Apt update] *********************************************************************************************************************************************
changed: [puppetdb]
changed: [puppetserver]
changed: [puppetagent_0]

PLAY [puppetserver] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [puppetserver]

TASK [Install Puppet Server] **********************************************************************************************************************************
ok: [puppetserver]

TASK [Change Java options Xms to reduce memory usage] *********************************************************************************************************
changed: [puppetserver]

TASK [Change Java Xmx to reduce memory usage] *****************************************************************************************************************
changed: [puppetserver]

TASK [Start and enable services] ******************************************************************************************************************************
ok: [puppetserver]

TASK [Check service status] ***********************************************************************************************************************************
changed: [puppetserver]

TASK [Show service status] ************************************************************************************************************************************
ok: [puppetserver] => {
    "result.stdout_lines": [
        "● puppetserver.service - puppetserver Service",
        "     Loaded: loaded (/lib/systemd/system/puppetserver.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 07:47:22 UTC; 24min ago",
        "    Process: 7812 ExecStart=/opt/puppetlabs/server/apps/puppetserver/bin/puppetserver start (code=exited, status=0/SUCCESS)",
        "   Main PID: 7863 (java)",
        "      Tasks: 52 (limit: 4915)",
        "     Memory: 578.3M",
        "        CPU: 38.067s",
        "     CGroup: /system.slice/puppetserver.service",
        "             └─7863 /usr/bin/java --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED -Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger -Dlogappender=F1 -XX:+CrashOnOutOfMemoryError -XX:ErrorFile=/var/log/puppetlabs/puppetserver/puppetserver_err_pid%p.log -cp \"/opt/puppetlabs/server/apps/puppetserver/puppet-server-release.jar:/opt/puppetlabs/server/data/puppetserver/jars/*\" clojure.main -m puppetlabs.trapperkeeper.main --config /etc/puppetlabs/puppetserver/conf.d --bootstrap-config /etc/puppetlabs/puppetserver/services.d/,/opt/puppetlabs/server/apps/puppetserver/config/services.d/ --restart-file /opt/puppetlabs/server/data/puppetserver/restartcounter",
        "",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,763 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@7d465d76 - The date pattern is 'yyyy-MM-dd' from file name pattern '/var/log/puppetlabs/puppetserver/puppetserver-access-%d{yyyy-MM-dd}.%i.log.gz'.",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,763 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@7d465d76 - Roll-over at midnight.",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,764 |-INFO in ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP@7d465d76 - Setting initial period to 2024-07-09T07:47:21.764Z",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,764 |-INFO in ch.qos.logback.core.model.processor.ImplicitModelHandler - Assuming default type [ch.qos.logback.access.PatternLayoutEncoder] for [encoder] property",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,767 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[FILE] - Active log file name: /var/log/puppetlabs/puppetserver/puppetserver-access.log",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,767 |-INFO in ch.qos.logback.core.rolling.RollingFileAppender[FILE] - File property is set to [/var/log/puppetlabs/puppetserver/puppetserver-access.log]",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,767 |-INFO in ch.qos.logback.core.model.processor.AppenderRefModelHandler - Attaching appender named [FILE] to null",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,767 |-INFO in ch.qos.logback.core.model.processor.DefaultProcessor@71ad9f59 - End of configuration.",
        "Jul 09 07:47:21 tf-puppetserver puppetserver[7863]: 07:47:21,767 |-INFO in ch.qos.logback.access.joran.JoranConfigurator@4676109d - Registering current configuration as safe fallback point",
        "Jul 09 07:47:22 tf-puppetserver systemd[1]: Started puppetserver Service."
    ]
}

PLAY [puppetagents] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [puppetagent_0]

TASK [Install Puppet Agent] ***********************************************************************************************************************************
ok: [puppetagent_0]

TASK [Avoid kernel warning popups] ****************************************************************************************************************************
changed: [puppetagent_0]

TASK [Avoid services need to be restarted warning popups] *****************************************************************************************************
changed: [puppetagent_0]

TASK [Start and enable services] ******************************************************************************************************************************
ok: [puppetagent_0]

TASK [Check service status] ***********************************************************************************************************************************
changed: [puppetagent_0]

TASK [Show service status] ************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "● puppet.service - Puppet agent",
        "     Loaded: loaded (/lib/systemd/system/puppet.service; enabled; vendor preset: enabled)",
        "     Active: active (running) since Tue 2024-07-09 08:06:55 UTC; 5min ago",
        "       Docs: man:puppet-agent(8)",
        "   Main PID: 8854 (puppet)",
        "      Tasks: 2 (limit: 2324)",
        "     Memory: 74.3M",
        "        CPU: 2.783s",
        "     CGroup: /system.slice/puppet.service",
        "             └─8854 /opt/puppetlabs/puppet/bin/ruby /opt/puppetlabs/puppet/bin/puppet agent --no-daemonize",
        "",
        "Jul 09 08:06:55 tf-puppetagent-0 systemd[1]: Started Puppet agent.",
        "Jul 09 08:06:57 tf-puppetagent-0 puppet-agent[8854]: Starting Puppet client version 8.7.0"
    ]
}

PLAY [puppetserver] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [puppetserver]

TASK [Sign CA from Puppet Agents] *****************************************************************************************************************************
changed: [puppetserver]

PLAY [puppetagents] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [puppetagent_0]

TASK [Test communication between Agents and Server] ***********************************************************************************************************
changed: [puppetagent_0]

TASK [Show test result] ***************************************************************************************************************************************
ok: [puppetagent_0] => {
    "result.stdout_lines": [
        "\u001b[0;32mInfo: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml\u001b[0m",
        "\u001b[0;32mInfo: Creating a new SSL certificate request for tf-puppetagent-0.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Certificate Request fingerprint (SHA256): D9:E7:6D:DD:9E:BC:F4:12:F0:4B:DA:7F:07:16:81:8E:1F:8E:7E:C9:22:92:EB:91:EC:92:45:77:C7:56:AF:93\u001b[0m",
        "\u001b[0;32mInfo: Downloaded certificate for tf-puppetagent-0.openstacklocal from https://puppet:8140/puppet-ca/v1\u001b[0m",
        "\u001b[0;32mInfo: Using environment 'production'\u001b[0m",
        "\u001b[0;32mInfo: Retrieving pluginfacts\u001b[0m",
        "\u001b[0;32mInfo: Retrieving plugin\u001b[0m",
        "\u001b[mNotice: Requesting catalog from puppet:8140 (10.100.65.79)\u001b[0m",
        "\u001b[mNotice: Catalog compiled by tf-puppetserver.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Caching catalog for tf-puppetagent-0.openstacklocal\u001b[0m",
        "\u001b[0;32mInfo: Applying configuration version '1720512742'\u001b[0m",
        "\u001b[0;32mInfo: Creating state file /opt/puppetlabs/puppet/cache/state/state.yaml\u001b[0m",
        "\u001b[mNotice: Applied catalog in 0.01 seconds\u001b[0m"
    ]
}

PLAY RECAP ****************************************************************************************************************************************************
puppetagent_0              : ok=15   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetdb                   : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
puppetserver               : ok=14   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
