# Lab1: Ansible inventory e primo playbook
##Soluzione esercizi
---

Questo documento vuole essere un supporto per la verifica in autonomia di quanto svolto durante gli esercizi proposti.

---

>**Esercizio 1**
>Facendo riferimento al file `labs/lab1/inventory.ini` e alla documentazione [Patterns: targeting hosts and groups](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html), identificare e provare con il parametro --list-hosts gli host che soddisfano i seguenti criteri:
>
>1. Tutti i web server
```
[vagrant@ansible lab1]$ ansible --list-hosts 'webservers'
  hosts (3):
    tweb01.example.com
    pweb01.example.com
    pweb02.example.com
[vagrant@ansible lab1]$ 
```
>2. Tutti i server di produzione
```
[vagrant@ansible lab1]$ ansible --list-hosts 'prod'
  hosts (4):
    pweb01.example.com
    pweb02.example.com
    pdb01.example.com
    pdb02.example.com
[vagrant@ansible lab1]$ 
```
>3. Tutti i web server di produzione

```
[vagrant@ansible lab1]$ ansible --list-hosts 'webservers:&prod'
  hosts (2):
    pweb01.example.com
    pweb02.example.com
[vagrant@ansible lab1]$ 
```
>4. Tutti i server che non sono di test
```
[vagrant@ansible lab1]$ ansible --list-hosts 'all:!test'
  hosts (5):
    lb.example.com
    pweb01.example.com
    pweb02.example.com
    pdb01.example.com
    pdb02.example.com
[vagrant@ansible lab1]$ 
```
>5. Tutti i server che non sono di test e non sono db server
```
[vagrant@ansible lab1]$ ansible --list-hosts 'all:!test:!dbservers'
  hosts (3):
    lb.example.com
    pweb01.example.com
    pweb02.example.com
[vagrant@ansible lab1]$ 
```
---
>**Esercizio 2**
>Facendo riferimento alla documentazione dei moduli [ansible.builtin.yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html) e [ansible.builtin.service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html):
> 1. creare il playbook `lab1/install_webserver.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>     * installazione del pacchetto **httpd**
>     * abilitazione del servizio **httpd** al boot e avvio del servizio
```yaml
---
- name: Install apache httpd
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present
      
    - name: Enable and start httpd service
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: started
```


> 2. verificare che il playbook sia sintatticamente corretto
```
[vagrant@ansible lab1]$ ansible-playbook --syntax-check install_webserver.yml 

playbook: install_webserver.yml
[vagrant@ansible lab1]$ 
```
> 3. eseguire il playbook
```
[vagrant@ansible lab1]$ ansible-playbook install_webserver.yml 

PLAY [Install apache httpd] *****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
changed: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab1]$ 
```
> 4. eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo
```
[vagrant@ansible lab1]$ curl http://pweb01.example.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html lang="en">
...
...
</html>
[vagrant@ansible lab1]$ 
```

> 5. rieseguire il playbook, osservare e commentare l'output
```
[vagrant@ansible lab1]$ ansible-playbook install_webserver.yml 

PLAY [Install apache httpd] *****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
ok: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab1]$ 
```
> 6. eseguire il comando `ssh pweb01 'sudo systemctl stop httpd'` e rieseguire il playbook
```
[vagrant@ansible lab1]$ ssh pweb01 'sudo systemctl stop httpd'
[vagrant@ansible lab1]$ ansible-playbook install_webserver.yml

PLAY [Install apache httpd] *****************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab1]$ 
```
> 7. osservare l'output delle tre esecuzioni del playbook e commentare

>La prima esecuzione ha eseguito gli step definiti nel playbook, installando il pacchetto e avviando il servizio.
Durante la seconda esecuzione, il pacchetto risulta già installato e il servizio già avviato, sembrerebbe che nessuno dei due task sia stato eseguito.
In realtà tale affermazione è errata, i task sono stati eseguiti ma non hanno apportato cambiamenti al sistema target in quanto il sistema era già nello "stato" configurato nel playbook (pacchetto httpd installato e servizio httpd abilitato e avviato).
Questo si evince dall'esito "ok" dello stato, che non risulta quindi "skipped".
>
>La terza esecuzione è stata avviata dopo aver stoppato manualmente il servizio httpd, lo stato del sistema target non corrisponde allo stato configurato nel playbook (servizio httpd attivo), il task "Enable and start httpd service" riporta quindi un esito "changed" in quanto ha eseguito un cambiamento sul sistema per riportarlo allo stato configurato nel playbook.

---

