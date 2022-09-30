# Lab4: Esecuzione condizionata dei task
##Soluzione esercizi
---

Questo documento vuole essere un supporto per la verifica in autonomia di quanto svolto durante gli esercizi proposti.

---
>**Esercizio 1**
>1. Modificare il playbook in modo da:
>     * Modificare il task YUM in modo che sia eseguito solo su piattaforme RedHat.
>     * Aggiungere un task per l'installazione del package https su piattaforme Debian, in modo che sia eseguito solo su piattaforme Debian.
>     * Aggiungere ai task che impattano sulla configurazione dell'http server la registrazione di una variabile di output.
>     * Subito dopo ogni task in cui è configurata la registrazione di una variabile di output, aggiungere un task per loggare, quando il playbook è eseguito in verbose, il valore della variabile registrata.
```yaml
---
# file: install_webserver.yml
# Play 1: Enable apache httpd service
#         Install httpd package, enable and start httpd service

- name: Enable apache httpd service
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package on RedHat
      when: ansible_os_family == "RedHat"
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Install httpd package on Debian
      when: ansible_os_family == "Debian"
      ansible.builtin.apt:
        name: httpd
        state: present

    - name: Configure & copy httpd.conf
      register: copy_conf_output
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf

    - name: Debug copy_conf_output
      debug:
        var: copy_conf_output
        verbosity: 1

    - name: Set permissions on /etc/httpd/conf/httpd.conf
      register: permissions_output
      ansible.builtin.file:
        path: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '644'

    - name: Debug permissions_output
      debug:
        var: permissions_output
        verbosity: 1

    - name: Copy index.html
      register: copy_index_output
      ansible.builtin.copy:
        content: 'Welcome to my Web Server {{ ansible_fqdn }} - {{ ansible_eth0.ipv4.address }}'
        dest: /usr/share/httpd/noindex/index.html

    - name: Debug copy_index_output
      debug:
        var: copy_index_output
        verbosity: 1

    - name: Enable and restart httpd service 
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: restarted
```
>2. Eseguire il playbook con il parametro "-v" ed osservare il log di esecuzione.
```
[vagrant@ansible lab4]$ ansible-playbook install_webserver.yml -v
Using /home/vagrant/labs/lab4/ansible.cfg as config file

PLAY [Enable apache httpd service] **************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on RedHat] **********************************************************************************************************************************************************
ok: [pweb01.example.com] => {"changed": false, "msg": "Nothing to do", "rc": 0, "results": []}

TASK [Install httpd package on Debian] **********************************************************************************************************************************************************
skipping: [pweb01.example.com] => {"changed": false, "skip_reason": "Conditional result was False"}

TASK [Configure & copy httpd.conf] **************************************************************************************************************************************************************
ok: [pweb01.example.com] => {"changed": false, "checksum": "0edcaade3201cf8ead50824e2fa152d1b22c0fdc", "dest": "/etc/httpd/conf/httpd.conf", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/httpd/conf/httpd.conf", "secontext": "system_u:object_r:httpd_config_t:s0", "size": 11901, "state": "file", "uid": 0}

TASK [Debug copy_conf_output] *******************************************************************************************************************************************************************
ok: [pweb01.example.com] => {
    "copy_conf_output": {
        "changed": false,
        "checksum": "0edcaade3201cf8ead50824e2fa152d1b22c0fdc",
        "dest": "/etc/httpd/conf/httpd.conf",
        "diff": {
            "after": {
                "path": "/etc/httpd/conf/httpd.conf"
            },
            "before": {
                "path": "/etc/httpd/conf/httpd.conf"
            }
        },
        "failed": false,
        "gid": 0,
        "group": "root",
        "mode": "0644",
        "owner": "root",
        "path": "/etc/httpd/conf/httpd.conf",
        "secontext": "system_u:object_r:httpd_config_t:s0",
        "size": 11901,
        "state": "file",
        "uid": 0
    }
}

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ********************************************************************************************************************************************
ok: [pweb01.example.com] => {"changed": false, "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/httpd/conf/httpd.conf", "secontext": "system_u:object_r:httpd_config_t:s0", "size": 11901, "state": "file", "uid": 0}

TASK [Debug permissions_output] *****************************************************************************************************************************************************************
ok: [pweb01.example.com] => {
    "permissions_output": {
        "changed": false,
        "diff": {
            "after": {
                "path": "/etc/httpd/conf/httpd.conf"
            },
            "before": {
                "path": "/etc/httpd/conf/httpd.conf"
            }
        },
        "failed": false,
        "gid": 0,
        "group": "root",
        "mode": "0644",
        "owner": "root",
        "path": "/etc/httpd/conf/httpd.conf",
        "secontext": "system_u:object_r:httpd_config_t:s0",
        "size": 11901,
        "state": "file",
        "uid": 0
    }
}

TASK [Copy index.html] **************************************************************************************************************************************************************************
ok: [pweb01.example.com] => {"changed": false, "checksum": "0fc6902e68b0ee6c2f1feccedcd0ecdae7a6cba0", "dest": "/usr/share/httpd/noindex/index.html", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/usr/share/httpd/noindex/index.html", "secontext": "unconfined_u:object_r:usr_t:s0", "size": 55, "state": "file", "uid": 0}

TASK [Debug copy_index_output] ******************************************************************************************************************************************************************
ok: [pweb01.example.com] => {
    "copy_index_output": {
        "changed": false,
        "checksum": "0fc6902e68b0ee6c2f1feccedcd0ecdae7a6cba0",
        "dest": "/usr/share/httpd/noindex/index.html",
        "diff": {
            "after": {
                "path": "/usr/share/httpd/noindex/index.html"
            },
            "before": {
                "path": "/usr/share/httpd/noindex/index.html"
            }
        },
        "failed": false,
        "gid": 0,
        "group": "root",
        "mode": "0644",
        "owner": "root",
        "path": "/usr/share/httpd/noindex/index.html",
        "secontext": "unconfined_u:object_r:usr_t:s0",
        "size": 55,
        "state": "file",
        "uid": 0
    }
}

TASK [Enable and restart httpd service] *********************************************************************************************************************************************************
changed: [pweb01.example.com] => {"changed": true, "enabled": true, "name": "httpd", "state": "started", "status": {"ActiveEnterTimestamp": "Fri 2022-09-16 07:27:00 UTC", "ActiveEnterTimestampMonotonic": "80054693363", "ActiveExitTimestamp": "Fri 2022-09-16 07:26:58 UTC", "ActiveExitTimestampMonotonic": "80053153634", "ActiveState": "active", "After": "system.slice nss-lookup.target tmp.mount basic.target httpd-init.service remote-fs.target systemd-journald.socket network.target sysinit.target -.mount systemd-tmpfiles-setup.service", "AllowIsolate": "no", "AllowedCPUs": "", "AllowedMemoryNodes": "", "AmbientCapabilities": "", "AssertResult": "yes", "AssertTimestamp": "Fri 2022-09-16 07:26:59 UTC", "AssertTimestampMonotonic": "80054331533", "Before": "multi-user.target shutdown.target", "BlockIOAccounting": "no", "BlockIOWeight": "[not set]", "CPUAccounting": "no", "CPUAffinity": "", "CPUAffinityFromNUMA": "no", "CPUQuotaPerSecUSec": "infinity", "CPUQuotaPeriodUSec": "infinity", "CPUSchedulingPolicy": "0", "CPUSchedulingPriority": "0", "CPUSchedulingResetOnFork": "no", "CPUShares": "[not set]", "CPUUsageNSec": "[not set]", "CPUWeight": "[not set]", "CacheDirectoryMode": "0755", "CanFreeze": "yes", "CanIsolate": "no", "CanReload": "yes", "CanStart": "yes", "CanStop": "yes", "CapabilityBoundingSet": "cap_chown cap_dac_override cap_dac_read_search cap_fowner cap_fsetid cap_kill cap_setgid cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config cap_mknod cap_lease cap_audit_write cap_audit_control cap_setfcap cap_mac_override cap_mac_admin cap_syslog cap_wake_alarm cap_block_suspend cap_audit_read cap_perfmon", "CollectMode": "inactive", "ConditionResult": "yes", "ConditionTimestamp": "Fri 2022-09-16 07:26:59 UTC", "ConditionTimestampMonotonic": "80054331533", "ConfigurationDirectoryMode": "0755", "Conflicts": "shutdown.target", "ControlGroup": "/system.slice/httpd.service", "ControlPID": "0", "DefaultDependencies": "yes", "DefaultMemoryLow": "0", "DefaultMemoryMin": "0", "Delegate": "no", "Description": "The Apache HTTP Server", "DevicePolicy": "auto", "Documentation": "man:httpd.service(8)", "DynamicUser": "no", "EffectiveCPUs": "", "EffectiveMemoryNodes": "", "Environment": "LANG=C", "ExecMainCode": "0", "ExecMainExitTimestampMonotonic": "0", "ExecMainPID": "68222", "ExecMainStartTimestamp": "Fri 2022-09-16 07:26:59 UTC", "ExecMainStartTimestampMonotonic": "80054333123", "ExecMainStatus": "0", "ExecReload": "{ path=/usr/sbin/httpd ; argv[]=/usr/sbin/httpd $OPTIONS -k graceful ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", "ExecStart": "{ path=/usr/sbin/httpd ; argv[]=/usr/sbin/httpd $OPTIONS -DFOREGROUND ; ignore_errors=no ; start_time=[Fri 2022-09-16 07:26:59 UTC] ; stop_time=[n/a] ; pid=68222 ; code=(null) ; status=0/0 }", "FailureAction": "none", "FileDescriptorStoreMax": "0", "FragmentPath": "/usr/lib/systemd/system/httpd.service", "FreezerState": "running", "GID": "[not set]", "GuessMainPID": "yes", "IOAccounting": "no", "IOSchedulingClass": "0", "IOSchedulingPriority": "0", "IOWeight": "[not set]", "IPAccounting": "no", "IPEgressBytes": "18446744073709551615", "IPEgressPackets": "18446744073709551615", "IPIngressBytes": "18446744073709551615", "IPIngressPackets": "18446744073709551615", "Id": "httpd.service", "IgnoreOnIsolate": "no", "IgnoreSIGPIPE": "yes", "InactiveEnterTimestamp": "Fri 2022-09-16 07:26:59 UTC", "InactiveEnterTimestampMonotonic": "80054330288", "InactiveExitTimestamp": "Fri 2022-09-16 07:26:59 UTC", "InactiveExitTimestampMonotonic": "80054333209", "InvocationID": "095e6bddb49b4424b1bd969b58834ca7", "JobRunningTimeoutUSec": "infinity", "JobTimeoutAction": "none", "JobTimeoutUSec": "infinity", "KeyringMode": "private", "KillMode": "mixed", "KillSignal": "28", "LimitAS": "infinity", "LimitASSoft": "infinity", "LimitCORE": "infinity", "LimitCORESoft": "infinity", "LimitCPU": "infinity", "LimitCPUSoft": "infinity", "LimitDATA": "infinity", "LimitDATASoft": "infinity", "LimitFSIZE": "infinity", "LimitFSIZESoft": "infinity", "LimitLOCKS": "infinity", "LimitLOCKSSoft": "infinity", "LimitMEMLOCK": "65536", "LimitMEMLOCKSoft": "65536", "LimitMSGQUEUE": "819200", "LimitMSGQUEUESoft": "819200", "LimitNICE": "0", "LimitNICESoft": "0", "LimitNOFILE": "262144", "LimitNOFILESoft": "1024", "LimitNPROC": "710", "LimitNPROCSoft": "710", "LimitRSS": "infinity", "LimitRSSSoft": "infinity", "LimitRTPRIO": "0", "LimitRTPRIOSoft": "0", "LimitRTTIME": "infinity", "LimitRTTIMESoft": "infinity", "LimitSIGPENDING": "710", "LimitSIGPENDINGSoft": "710", "LimitSTACK": "infinity", "LimitSTACKSoft": "8388608", "LoadState": "loaded", "LockPersonality": "no", "LogLevelMax": "-1", "LogRateLimitBurst": "0", "LogRateLimitIntervalUSec": "0", "LogsDirectoryMode": "0755", "MainPID": "68222", "MemoryAccounting": "yes", "MemoryCurrent": "13512704", "MemoryDenyWriteExecute": "no", "MemoryHigh": "infinity", "MemoryLimit": "infinity", "MemoryLow": "0", "MemoryMax": "infinity", "MemoryMin": "0", "MemorySwapMax": "infinity", "MountAPIVFS": "no", "MountFlags": "", "NFileDescriptorStore": "0", "NRestarts": "0", "NUMAMask": "", "NUMAPolicy": "n/a", "Names": "httpd.service", "NeedDaemonReload": "no", "Nice": "0", "NoNewPrivileges": "no", "NonBlocking": "no", "NotifyAccess": "main", "OOMScoreAdjust": "0", "OnFailureJobMode": "replace", "PermissionsStartOnly": "no", "Perpetual": "no", "PrivateDevices": "no", "PrivateMounts": "no", "PrivateNetwork": "no", "PrivateTmp": "yes", "PrivateUsers": "no", "ProtectControlGroups": "no", "ProtectHome": "no", "ProtectKernelModules": "no", "ProtectKernelTunables": "no", "ProtectSystem": "no", "RefuseManualStart": "no", "RefuseManualStop": "no", "RemainAfterExit": "no", "RemoveIPC": "no", "Requires": "system.slice sysinit.target -.mount", "RequiresMountsFor": "/var/tmp", "Restart": "no", "RestartUSec": "100ms", "RestrictNamespaces": "no", "RestrictRealtime": "no", "RestrictSUIDSGID": "no", "Result": "success", "RootDirectoryStartOnly": "no", "RuntimeDirectoryMode": "0755", "RuntimeDirectoryPreserve": "no", "RuntimeMaxUSec": "infinity", "SameProcessGroup": "no", "SecureBits": "0", "SendSIGHUP": "no", "SendSIGKILL": "yes", "Slice": "system.slice", "StandardError": "inherit", "StandardInput": "null", "StandardInputData": "", "StandardOutput": "journal", "StartLimitAction": "none", "StartLimitBurst": "5", "StartLimitIntervalUSec": "10s", "StartupBlockIOWeight": "[not set]", "StartupCPUShares": "[not set]", "StartupCPUWeight": "[not set]", "StartupIOWeight": "[not set]", "StateChangeTimestamp": "Fri 2022-09-16 07:27:00 UTC", "StateChangeTimestampMonotonic": "80054738739", "StateDirectoryMode": "0755", "StatusErrno": "0", "StatusText": "Running, listening on: port 8080", "StopWhenUnneeded": "no", "SubState": "running", "SuccessAction": "none", "SyslogFacility": "3", "SyslogLevel": "6", "SyslogLevelPrefix": "yes", "SyslogPriority": "30", "SystemCallErrorNumber": "0", "TTYReset": "no", "TTYVHangup": "no", "TTYVTDisallocate": "no", "TasksAccounting": "yes", "TasksCurrent": "213", "TasksMax": "1136", "TimeoutStartUSec": "1min 30s", "TimeoutStopUSec": "1min 30s", "TimerSlackNSec": "50000", "Transient": "no", "Type": "notify", "UID": "[not set]", "UMask": "0022", "UnitFilePreset": "disabled", "UnitFileState": "enabled", "UtmpMode": "init", "WantedBy": "multi-user.target", "Wants": "httpd-init.service", "WatchdogTimestamp": "Fri 2022-09-16 07:27:00 UTC", "WatchdogTimestampMonotonic": "80054693360", "WatchdogUSec": "0"}}

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=9    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```
>3. Modificare il task di riavvio del servizio in modo che sia eseguito solo quando almeno uno dei task modificati al punto 1 risulti changed.
```yaml
---
# file: install_webserver.yml
# Play 1: Enable apache httpd service
#         Install httpd package, enable and start httpd service

- name: Enable apache httpd service
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package on RedHat
      when: ansible_os_family == "RedHat"
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Install httpd package on Debian
      when: ansible_os_family == "Debian"
      ansible.builtin.apt:
        name: httpd
        state: present

    - name: Configure & copy httpd.conf
      register: copy_conf_output
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf

    - name: Debug copy_conf_output
      debug:
        var: copy_conf_output
        verbosity: 1

    - name: Set permissions on /etc/httpd/conf/httpd.conf
      register: permissions_output
      ansible.builtin.file:
        path: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '644'

    - name: Debug permissions_output
      debug:
        var: permissions_output
        verbosity: 1

    - name: Copy index.html
      register: copy_index_output
      ansible.builtin.copy:
        content: 'Welcome to my Web Server {{ ansible_fqdn }} - {{ ansible_eth0.ipv4.address }}'
        dest: /usr/share/httpd/noindex/index.html

    - name: Debug copy_index_output
      debug:
        var: copy_index_output
        verbosity: 1

    - name: Enable and restart httpd service 
      when: copy_conf_output.changed or permissions_output.changed or copy_index_output.changed
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: restarted
```
>4. Eseguire il playbook e verificare che il servizio non sia stato riavviato.
```
[vagrant@ansible lab4]$ ansible-playbook install_webserver.yml 

PLAY [Enable apache httpd service] **************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on RedHat] **********************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on Debian] **********************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Debug copy_conf_output] *******************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ********************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Debug permissions_output] *****************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Copy index.html] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Debug copy_index_output] ******************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Enable and restart httpd service] *********************************************************************************************************************************************************
skipping: [pweb01.example.com]

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=5    changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   

```
>5. Eseguire il comando `curl http://pweb01.example.com:8080` per verificare che il server http sia attivo sulla porta 8080. (configurata nell'inventory)
```
[vagrant@ansible lab4]$ curl http://pweb01.example.com:8080
Welcome to my Web Server pweb01.example.com - 10.0.2.15
```
>6. Eseguire il playbook passando la variabile http_port=8888 come extra vars e dal log di esecuzione verificare che il servizio sia stato riavviato.
```
[vagrant@ansible lab4]$ ansible-playbook install_webserver.yml --extra-vars http_port=8888

PLAY [Enable apache httpd service] **************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on RedHat] **********************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on Debian] **********************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **************************************************************************************************************************************************************
changed: [pweb01.example.com]

TASK [Debug copy_conf_output] *******************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ********************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Debug permissions_output] *****************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Copy index.html] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Debug copy_index_output] ******************************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Enable and restart httpd service] *********************************************************************************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=6    changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   

```
>7. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo sulla porta 8888.
```
[vagrant@ansible lab4]$ curl http://pweb01.example.com:8888
Welcome to my Web Server pweb01.example.com - 10.0.2.15
```

---

>**Esercizio 2**
>1. Modificare il playbook in modo da:
>     * Spostare il task di riavvio del servizio nella sezione handlers ed eliminare le condizioni `when:`.
>     * Rimuovere la registrazione delle variabili di output e i task ansible.builtin.debug. 
>     * Aggiungere ai task che richiedono il riavvio del servizio, la notifica all'handler.
```yaml
---
# file: install_webserver.yml
# Play 1: Enable apache httpd service
#         Install httpd package, enable and start httpd service

- name: Enable apache httpd service
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package on RedHat
      when: ansible_os_family == "RedHat"
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Install httpd package on Debian
      when: ansible_os_family == "Debian"
      ansible.builtin.apt:
        name: httpd
        state: present

    - name: Configure & copy httpd.conf
      notify: Enable and restart httpd service
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf

    - name: Set permissions on /etc/httpd/conf/httpd.conf
      notify: Enable and restart httpd service
      ansible.builtin.file:
        path: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '644'

    - name: Copy index.html
      notify: Enable and restart httpd service
      ansible.builtin.copy:
        content: 'Welcome to my Web Server {{ ansible_fqdn }} - {{ ansible_eth0.ipv4.address }}'
        dest: /usr/share/httpd/noindex/index.html

  handlers:
    - name: Enable and restart httpd service 
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: restarted
```
>2. Eseguire il playbook e verificare che il servizio sia stato riavviato.
```
[vagrant@ansible lab4]$ ansible-playbook install_webserver.yml 

PLAY [Enable apache httpd service] **************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on RedHat] **********************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on Debian] **********************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **************************************************************************************************************************************************************
changed: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ********************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

RUNNING HANDLER [Enable and restart httpd service] **********************************************************************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```
>3. Eseguire nuovamente il playbook e verificare che il servizio non sia stato riavviato.
```
[vagrant@ansible lab4]$ ansible-playbook install_webserver.yml 

PLAY [Enable apache httpd service] **************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on RedHat] **********************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package on Debian] **********************************************************************************************************************************************************
skipping: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **************************************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ********************************************************************************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **************************************************************************************************************************************************************************
ok: [pweb01.example.com]

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```

---

