# Lab0: Installazione ansible sul control node
---

**Obiettivo:** Predisporre il control node e i managed host all'esecuzione dei playbook Ansible.

Nel terminale di VSCode già aperto sul host ansible verifichiamo che:
* i managed host siano configurati correttamente nel file /etc/hosts
* la chiave privata dell'utente ansible sia trustata sui managed host, abilitando la connessione senza richiesta di password (verifica limitata al solo managed host attivo in questo momento)

```
[vagrant@ansible ~]$ cat /etc/hosts
192.168.60.10   ansible.example.com     ansible
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.60.100  lb lb.example.com
192.168.60.20  pweb01 pweb01.example.com
192.168.60.21  pweb02 pweb02.example.com
192.168.60.30  pdb01 pdb01.example.com
192.168.60.31  pdb02 pdb02.example.com
[vagrant@ansible ~]$ ssh pweb01.example.com
Warning: Permanently added 'pweb01.example.com,192.168.60.20' (ECDSA) to the list of known hosts.
[vagrant@pweb01 ~]$ exit
logout
Connection to pweb01 closed.
[vagrant@ansible ~]$ 
```

**Installazione ansible sul control node**
Procedere con l'installazione di ansible sul control node, usando il package manager in modo da installare automaticamente anche eventuali dipendenze, ad esempio la versione di python richiesta, non presenti sul sistema.
Nel nostro caso useremo yum (richiede i privilegi di root quindi dovrà essere eseguito con **sudo**) per installare il package ansible-core.

```
[vagrant@ansible ~]$ sudo yum install ansible-core
Last metadata expiration check: 0:04:55 ago on Thu 15 Sep 2022 09:52:14 AM UTC.
Dependencies resolved.
=============================================================================================================================
 Package                            Architecture    Version                                         Repository          Size
=============================================================================================================================
Installing:
 ansible-core                       x86_64          2.13.3-1.el8                                    appstream          2.8 M
Installing dependencies:
 git-core                           x86_64          2.31.1-2.el8                                    appstream          4.7 M
 python39                           x86_64          3.9.6-1.module_el8.5.0+872+ab54e8e5             appstream           32 k
 ...
 ...
 Transaction Summary
=============================================================================================================================
Install  16 Packages

Total download size: 22 M
Installed size: 87 M
Is this ok [y/N]: y
...
...
Complete!
[vagrant@ansible ~]$ 
```


**Verifica requisiti dei managed host**
Ogni managed host deve soddisfare i seguenti requisiti affinchè possa essere gestito con ansible:

* Python >= 2.6 or >= 3.5
* SSH, sftp for file transfer or scp if not available, on UNIK-like machine
* WinRM on Windows machine

Tipicamente tali requisiti saranno indirizzati in fase di provisioning dei sistemi.

Per verificare tali requisiti è possibile usare il modulo ansible `ansible.builtin.ping`, se il modulo viene eseguito correttamente allora è possibile gestire l'host target con ansible.
Come tutti i moduli ansible, anche il modulo `ping` può essere usato come un **ansible ad hoc command** ([Introduction to ad hoc commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)). Tale modalità è comoda per l'esecuzione di comandi singoli su uno o su tutti gli host presenti nell'inventory.

Spostarsi nella directory labs/lab0 e provare ad eseguire il modulo ping per l'host `pweb01.example.com`.

```
[vagrant@ansible ~]$ cd labs/lab0
[vagrant@ansible lab0]$ ansible -m ansible.builtin.ping pweb01.example.com
pweb01.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
[vagrant@ansible lab0]
```

Possiamo quindi gestire con ansible l'host `pweb01.example.com`.

---