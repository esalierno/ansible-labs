# Lab2: Variabili e fact 
##Soluzione esercizi
---

Questo documento vuole essere un supporto per la verifica in autonomia di quanto svolto durante gli esercizi proposti.

---

>**Esercizio 1**
>Facendo riferimento alla tabella seguente, che riporta i valori da assegnare alla variabile http_port con le modalità indicate:
>1. creare il playbook `lab2/print_http_port.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>    * print della variabile http_port
>    * impostazione del fact http_port
>    * print della variabile http_port
```yaml
---
- name: Print http_port value
  hosts: 'pweb01.example.com'

  tasks:
    - name: Print variable http_port
      ansible.builtin.debug:
        var: http_port

    - name: Setting host fact http_port
      ansible.builtin.set_fact:
        http_port: 8083

    - name: Print variable http_port
      ansible.builtin.debug:
        var: http_port
```
>2. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
```
[vagrant@ansible lab2]$ ansible-playbook print_http_port.yml 

PLAY [Print http_port value] ****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8080
}

TASK [Setting host fact http_port] **********************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8083
}

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab2]$ 
```
>3. impostare nel file `lab2/inventory.ini` la variabile http_port sull'host `pweb01.example.com`
```ini
lb.example.com

[webservers]
tweb01.example.com ansible_host=10.0.2.10
pweb01.example.com http_port=8081
pweb02.example.com

[dbservers]
tdb01.example.com
pdb01.example.com
pdb02.example.com

[test]
tweb01.example.com
tdb01.example.com

[prod]
pweb01.example.com
pweb02.example.com
pdb01.example.com
pdb02.example.com

[webservers:vars]
http_port=8080
```
>4. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
```
[vagrant@ansible lab2]$ ansible-playbook print_http_port.yml 

PLAY [Print http_port value] ****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8081
}

TASK [Setting host fact http_port] **********************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8083
}

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab2]$ 
```
>5. modificare il playbook `lab2/print_http_port.yml` impostando la variabile http_port nelle `play vars`
```yaml
---
- name: Print http_port value
  hosts: 'pweb01.example.com'
  vars:
    http_port: 8082

  tasks:
    - name: Print variable http_port
      ansible.builtin.debug:
        var: http_port

    - name: Setting host fact http_port
      ansible.builtin.set_fact:
        http_port: 8083

    - name: Print variable http_port
      ansible.builtin.debug:
        var: http_port
```
>6. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
```
[vagrant@ansible lab2]$ ansible-playbook print_http_port.yml 

PLAY [Print http_port value] ****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8082
}

TASK [Setting host fact http_port] **********************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": 8083
}

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab2]$ 
```
>7. eseguire il playbook impostando la variabile http_port come extra vars e verificare i valori della variabile nel log di esecuzione
```
[vagrant@ansible lab2]$ ansible-playbook --extra-vars http_port=8084 print_http_port.yml

PLAY [Print http_port value] ****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": "8084"
}

TASK [Setting host fact http_port] **********************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "http_port": "8084"
}

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab2]$ 
```

---

>**Esercizio 2**
>E' possibile vedere i fact recuperati da un sistema remoto con il modulo [ansible.builtin.setup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html). Eseguire il modulo `ansible.builtin.setup` come un ad-hoc command verso l'host `pweb01.example.com`
```
[vagrant@ansible lab2]$ ansible -m ansible.builtin.setup pweb01.example.com
pweb01.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.60.20"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::5054:ff:fe03:15fa",
            "fe80::a00:27ff:fe3e:a91"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_vendor": "innotek GmbH",
        "ansible_bios_version": "VirtualBox",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "VirtualBox",
        "ansible_board_serial": "NA",
        "ansible_board_vendor": "Oracle Corporation",
        "ansible_board_version": "1.2",
        "ansible_chassis_asset_tag": "NA",
        "ansible_chassis_serial": "NA",
        "ansible_chassis_vendor": "Oracle Corporation",
        "ansible_chassis_version": "NA",
        "ansible_cmdline": {
            "BOOT_IMAGE": "(hd0,msdos1)/boot/vmlinuz-4.18.0-277.el8.x86_64",
            "biosdevname": "0",
            "console": "ttyS0,115200n8",
            "elevator": "noop",
            "net.ifnames": "0",
            "no_timer_check": true,
            "ro": true,
            "root": "UUID=ea09066e-02dd-46ad-bac9-700172bc3bca"
        },
        "ansible_date_time": {
            "date": "2022-09-14",
            "day": "14",
            "epoch": "1663167001",
            "epoch_int": "1663167001",
            "hour": "14",
            "iso8601": "2022-09-14T14:50:01Z",
            "iso8601_basic": "20220914T145001343915",
            "iso8601_basic_short": "20220914T145001",
            "iso8601_micro": "2022-09-14T14:50:01.343915Z",
            "minute": "50",
            "month": "09",
            "second": "01",
            "time": "14:50:01",
            "tz": "UTC",
            "tz_dst": "UTC",
            "tz_offset": "+0000",
            "weekday": "Thursday",
            "weekday_number": "4",
            "weeknumber": "37",
            "year": "2022"
        },
        "ansible_default_ipv4": {
            "address": "10.0.2.15",
            "alias": "eth0",
            "broadcast": "10.0.2.255",
            "gateway": "10.0.2.2",
            "interface": "eth0",
            "macaddress": "52:54:00:03:15:fa",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "10.0.2.0",
            "prefix": "24",
            "type": "ether"
        },
        "ansible_default_ipv6": {},
        "ansible_device_links": {
            "ids": {
                "sda": [
                    "ata-VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                    "scsi-0ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                    "scsi-1ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                    "scsi-SATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6"
                ],
                "sda1": [
                    "ata-VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                    "scsi-0ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                    "scsi-1ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                    "scsi-SATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1"
                ]
            },
            "labels": {},
            "masters": {},
            "uuids": {
                "sda1": [
                    "ea09066e-02dd-46ad-bac9-700172bc3bca"
                ]
            }
        },
        "ansible_devices": {
            "sda": {
                "holders": [],
                "host": "IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)",
                "links": {
                    "ids": [
                        "ata-VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                        "scsi-0ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                        "scsi-1ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6",
                        "scsi-SATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VBOX HARDDISK",
                "partitions": {
                    "sda1": {
                        "holders": [],
                        "links": {
                            "ids": [
                                "ata-VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                                "scsi-0ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                                "scsi-1ATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1",
                                "scsi-SATA_VBOX_HARDDISK_VBdfc7319a-a7769dd6-part1"
                            ],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "ea09066e-02dd-46ad-bac9-700172bc3bca"
                            ]
                        },
                        "sectors": "20969472",
                        "sectorsize": 512,
                        "size": "10.00 GB",
                        "start": "2048",
                        "uuid": "ea09066e-02dd-46ad-bac9-700172bc3bca"
                    }
                },
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "mq-deadline",
                "sectors": "20971520",
                "sectorsize": "512",
                "size": "10.00 GB",
                "support_discard": "0",
                "vendor": "ATA",
                "virtual": 1
            }
        },
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/centos-release",
        "ansible_distribution_file_variety": "CentOS",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Stream",
        "ansible_distribution_version": "8",
        "ansible_dns": {
            "nameservers": [
                "10.0.2.3"
            ],
            "search": [
                "emea.ibm.com",
                "example.com"
            ]
        },
        "ansible_domain": "example.com",
        "ansible_effective_group_id": 1000,
        "ansible_effective_user_id": 1000,
        "ansible_env": {
            "DBUS_SESSION_BUS_ADDRESS": "unix:path=/run/user/1000/bus",
            "HOME": "/home/vagrant",
            "LANG": "en_US.UTF-8",
            "LESSOPEN": "||/usr/bin/lesspipe.sh %s",
            "LOGNAME": "vagrant",
            "LS_COLORS": "rs=0:di=38;5;33:ln=38;5;51:mh=00:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=01;05;37;41:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;40:*.tar=38;5;9:*.tgz=38;5;9:*.arc=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lha=38;5;9:*.lz4=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.tzo=38;5;9:*.t7z=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lrz=38;5;9:*.lz=38;5;9:*.lzo=38;5;9:*.xz=38;5;9:*.zst=38;5;9:*.tzst=38;5;9:*.bz2=38;5;9:*.bz=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.war=38;5;9:*.ear=38;5;9:*.sar=38;5;9:*.rar=38;5;9:*.alz=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.cab=38;5;9:*.wim=38;5;9:*.swm=38;5;9:*.dwm=38;5;9:*.esd=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.mjpg=38;5;13:*.mjpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.webm=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.m4a=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.oga=38;5;45:*.opus=38;5;45:*.spx=38;5;45:*.xspf=38;5;45:",
            "PATH": "/home/vagrant/.local/bin:/home/vagrant/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin",
            "PWD": "/home/vagrant",
            "SELINUX_LEVEL_REQUESTED": "",
            "SELINUX_ROLE_REQUESTED": "",
            "SELINUX_USE_CURRENT_RANGE": "",
            "SHELL": "/bin/bash",
            "SHLVL": "2",
            "SSH_CLIENT": "192.168.60.10 38596 22",
            "SSH_CONNECTION": "192.168.60.10 38596 192.168.60.20 22",
            "SSH_TTY": "/dev/pts/0",
            "TERM": "xterm-256color",
            "USER": "vagrant",
            "XDG_RUNTIME_DIR": "/run/user/1000",
            "XDG_SESSION_ID": "48",
            "_": "/usr/libexec/platform-python"
        },
        "ansible_eth0": {
            "active": true,
            "device": "eth0",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "off [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off",
                "rx_checksumming": "off",
                "rx_fcs": "off",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "off [fixed]",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "off [fixed]",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "10.0.2.15",
                "broadcast": "10.0.2.255",
                "netmask": "255.255.255.0",
                "network": "10.0.2.0",
                "prefix": "24"
            },
            "ipv6": [
                {
                    "address": "fe80::5054:ff:fe03:15fa",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "52:54:00:03:15:fa",
            "module": "e1000",
            "mtu": 1500,
            "pciid": "0000:00:03.0",
            "promisc": false,
            "speed": 1000,
            "timestamping": [],
            "type": "ether"
        },
        "ansible_eth1": {
            "active": true,
            "device": "eth1",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "off [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off",
                "rx_checksumming": "off",
                "rx_fcs": "off",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "off [fixed]",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "off [fixed]",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "192.168.60.20",
                "broadcast": "192.168.60.255",
                "netmask": "255.255.255.0",
                "network": "192.168.60.0",
                "prefix": "24"
            },
            "ipv6": [
                {
                    "address": "fe80::a00:27ff:fe3e:a91",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "08:00:27:3e:0a:91",
            "module": "e1000",
            "mtu": 1500,
            "pciid": "0000:00:08.0",
            "promisc": false,
            "speed": 1000,
            "timestamping": [],
            "type": "ether"
        },
        "ansible_fibre_channel_wwn": [],
        "ansible_fips": false,
        "ansible_form_factor": "Other",
        "ansible_fqdn": "pweb01.example.com",
        "ansible_hostname": "pweb01",
        "ansible_hostnqn": "",
        "ansible_interfaces": [
            "eth1",
            "lo",
            "eth0"
        ],
        "ansible_is_chroot": false,
        "ansible_iscsi_iqn": "",
        "ansible_kernel": "4.18.0-277.el8.x86_64",
        "ansible_kernel_version": "#1 SMP Wed Feb 3 20:35:19 UTC 2021",
        "ansible_lo": {
            "active": true,
            "device": "lo",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "on [fixed]",
                "netns_local": "on [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on [fixed]",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "off [fixed]",
                "rx_vlan_offload": "off [fixed]",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on [fixed]",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "on [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "on [fixed]",
                "tx_nocache_copy": "off [fixed]",
                "tx_scatter_gather": "on [fixed]",
                "tx_scatter_gather_fraglist": "on [fixed]",
                "tx_sctp_segmentation": "on",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "on",
                "tx_tcp_mangleid_segmentation": "on",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "off [fixed]",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "off [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "on [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "127.0.0.1",
                "broadcast": "",
                "netmask": "255.0.0.0",
                "network": "127.0.0.0",
                "prefix": "8"
            },
            "ipv6": [
                {
                    "address": "::1",
                    "prefix": "128",
                    "scope": "host"
                }
            ],
            "mtu": 65536,
            "promisc": false,
            "timestamping": [],
            "type": "loopback"
        },
        "ansible_local": {},
        "ansible_lsb": {},
        "ansible_machine": "x86_64",
        "ansible_machine_id": "104fddf1e25241efb42e70810d8cf700",
        "ansible_memfree_mb": 25,
        "ansible_memory_mb": {
            "nocache": {
                "free": 98,
                "used": 115
            },
            "real": {
                "free": 25,
                "total": 213,
                "used": 188
            },
            "swap": {
                "cached": 2,
                "free": 2003,
                "total": 2047,
                "used": 44
            }
        },
        "ansible_memtotal_mb": 213,
        "ansible_mounts": [
            {
                "block_available": 1705306,
                "block_size": 4096,
                "block_total": 2618624,
                "block_used": 913318,
                "device": "/dev/sda1",
                "fstype": "xfs",
                "inode_available": 5208404,
                "inode_total": 5242368,
                "inode_used": 33964,
                "mount": "/",
                "options": "rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota",
                "size_available": 6984933376,
                "size_total": 10725883904,
                "uuid": "ea09066e-02dd-46ad-bac9-700172bc3bca"
            }
        ],
        "ansible_nodename": "pweb01.example.com",
        "ansible_os_family": "RedHat",
        "ansible_pkg_mgr": "dnf",
        "ansible_proc_cmdline": {
            "BOOT_IMAGE": "(hd0,msdos1)/boot/vmlinuz-4.18.0-277.el8.x86_64",
            "biosdevname": "0",
            "console": [
                "tty0",
                "ttyS0,115200n8"
            ],
            "elevator": "noop",
            "net.ifnames": "0",
            "no_timer_check": true,
            "ro": true,
            "root": "UUID=ea09066e-02dd-46ad-bac9-700172bc3bca"
        },
        "ansible_processor": [
            "0",
            "GenuineIntel",
            "Intel(R) Core(TM) i5-8210Y CPU @ 1.60GHz"
        ],
        "ansible_processor_cores": 1,
        "ansible_processor_count": 1,
        "ansible_processor_nproc": 1,
        "ansible_processor_threads_per_core": 1,
        "ansible_processor_vcpus": 1,
        "ansible_product_name": "VirtualBox",
        "ansible_product_serial": "NA",
        "ansible_product_uuid": "NA",
        "ansible_product_version": "1.2",
        "ansible_python": {
            "executable": "/usr/libexec/platform-python",
            "has_sslcontext": true,
            "type": "cpython",
            "version": {
                "major": 3,
                "micro": 8,
                "minor": 6,
                "releaselevel": "final",
                "serial": 0
            },
            "version_info": [
                3,
                6,
                8,
                "final",
                0
            ]
        },
        "ansible_python_version": "3.6.8",
        "ansible_real_group_id": 1000,
        "ansible_real_user_id": 1000,
        "ansible_selinux": {
            "config_mode": "enforcing",
            "mode": "enforcing",
            "policyvers": 33,
            "status": "enabled",
            "type": "targeted"
        },
        "ansible_selinux_python_present": true,
        "ansible_service_mgr": "systemd",
        "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL8e6Xenwvsy82fnhdfFMZ9tdTPn9uNyE+s6jiXsKa9hAB3jxQN+8fmohVTfygc8T+Vil4oX1QEbDhVtpxFjx5Y=",
        "ansible_ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256",
        "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIIKMIrZHVvJ2nyzbNlTqkerCbzqw+cK+qo0CqMVuHCgE",
        "ansible_ssh_host_key_ed25519_public_keytype": "ssh-ed25519",
        "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABgQDD0JB8z2loBUk3Rc9EELM7mRS7ZwzJMHxcMRUHg2kb6IGju/vl8r8HTemuh7N/ikANKZQ6y3iCc0dafackRVumhVzJdWhY6DIt7dWj8ZZixoPoCAnxHCuVY54KW/47xxQ2n6RXkYFrtlDrYWz1HKGQEVOCk2hJnwhTGOJy8I8fb7TX9nd7JwRl2MxdX7cPXFixkYz2YZMlZFQBFgG1I/5pqJB4yNpj2T0wO+rtMQwxdAnqkyvzXRSHxWD0Ug7i7QXiDfVBkyZcVZiff7YWef0Mv2AM2yR6Hkn1vaAjQcImNCcHLm1YIZn0lDm+boZudaH8BnGWRy8oPQMjQweIBS/2M4qOayXK0X9MCkpHq10nh0sh03pKjxvXSI8KPp0fyyCrdrn2nAq/IQipW+ftniYjdoHAktfcwXhO1F0ClqXeAF8B3UldGjBZFkFpPdn1nHTXbSn3b5jvkk8MFYpJojgYCUQQfjX7pH+eXiNyqPy4MREZ4q3h4AuV82bFsHIrW5U=",
        "ansible_ssh_host_key_rsa_public_keytype": "ssh-rsa",
        "ansible_swapfree_mb": 2003,
        "ansible_swaptotal_mb": 2047,
        "ansible_system": "Linux",
        "ansible_system_capabilities": [
            ""
        ],
        "ansible_system_capabilities_enforced": "True",
        "ansible_system_vendor": "innotek GmbH",
        "ansible_uptime_seconds": 31035,
        "ansible_user_dir": "/home/vagrant",
        "ansible_user_gecos": "",
        "ansible_user_gid": 1000,
        "ansible_user_id": "vagrant",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 1000,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_tech_guest": [
            "virtualbox"
        ],
        "ansible_virtualization_tech_host": [],
        "ansible_virtualization_type": "virtualbox",
        "discovered_interpreter_python": "/usr/libexec/platform-python",
        "gather_subset": [
            "all"
        ],
        "module_setup": true
    },
    "changed": false
}
[vagrant@ansible lab2]$ 
```

---

>**Esercizio 3**
>1. creare il playbook `lab2/print_some_facts.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>    * print dell'hostname
>    * print della distribuzione del sistema operativo
>    * print della versione del sistema operativo
>    * print di tutti gli indirizzi ipv4
>    * print del solo indirizzo ipv4 della scheda eth0
```yaml
---
- name: Print http_port value
  hosts: 'pweb01.example.com'

  tasks:
    - name: Print variable ansible_hostname
      ansible.builtin.debug:
        var: ansible_hostname

    - name: Print variable ansible_distribution
      ansible.builtin.debug:
        var: ansible_distribution

    - name: Print variable ansible_distribution_version
      ansible.builtin.debug:
        var: ansible_distribution_version

    - name: Print variable http_port
      ansible.builtin.debug:
        var: ansible_all_ipv4_addresses

    - name: Print variable http_port
      ansible.builtin.debug:
        var: ansible_eth0.ipv4.address
```
>2. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
```
[vagrant@ansible lab2]$ ansible-playbook print_some_facts.yml 

PLAY [Print http_port value] ****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Print variable ansible_hostname] ******************************************************************************************
ok: [pweb01.example.com] => {
    "ansible_hostname": "pweb01"
}

TASK [Print variable ansible_distribution] **************************************************************************************
ok: [pweb01.example.com] => {
    "ansible_distribution": "CentOS"
}

TASK [Print variable ansible_distribution_version] ******************************************************************************
ok: [pweb01.example.com] => {
    "ansible_distribution_version": "8"
}

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "ansible_all_ipv4_addresses": [
        "10.0.2.15",
        "192.168.60.20"
    ]
}

TASK [Print variable http_port] *************************************************************************************************
ok: [pweb01.example.com] => {
    "ansible_eth0.ipv4.address": "10.0.2.15"
}

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab2]$ 
```

---

>**Esercizio 4**
>5. Eseguire più volte il modulo `ansible.builtin.setup` come ad-hoc command sul server `pweb01.example.com` e verificarne l'output.
```
[vagrant@ansible lab2]$ ansible -m ansible.builtin.setup -a filter='ansible_local' pweb01.example.com 
pweb01.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "custom": {
                "installed_applications": {
                    "gar": {
                        "name": "Gestione accertamenti e regolarizzazione",
                        "version": "2.4.5"
                    },
                    "ic": {
                        "name": "Istanze e comunicazioni ",
                        "version": "5.2.0"
                    },
                    "pr": {
                        "name": "Pagamenti e rimborsi",
                        "version": "1.1.0"
                    }
                }
            },
            "dyncustom": {
                "executiontime": "20220914-15.07.17"
            }
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
[vagrant@ansible lab2]$ ansible -m ansible.builtin.setup -a filter='ansible_local' pweb01.example.com 
pweb01.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "custom": {
                "installed_applications": {
                    "gar": {
                        "name": "Gestione accertamenti e regolarizzazione",
                        "version": "2.4.5"
                    },
                    "ic": {
                        "name": "Istanze e comunicazioni ",
                        "version": "5.2.0"
                    },
                    "pr": {
                        "name": "Pagamenti e rimborsi",
                        "version": "1.1.0"
                    }
                }
            },
            "dyncustom": {
                "executiontime": "20220914-15.07.37"
            }
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
[vagrant@ansible lab2]$ 
```

---
