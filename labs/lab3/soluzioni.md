# Lab3: Copia file e modifica file di configurazione
##Soluzione esercizi
---

Questo documento vuole essere un supporto per la verifica in autonomia di quanto svolto durante gli esercizi proposti.

---
>**Esercizio 1**
>1. Spostarsi nella directory `lab3` e creare la directory `files` sul control node.
```
[vagrant@ansible lab3]$ mkdir files
[vagrant@ansible lab3]$ 
```
>2. Usando il comando Linux `scp <host>:<path file remoto> <path locale>`, recuperare dal managed host `pweb01.example.com` il file `/etc/httpd/conf/httpd.conf` e copiarlo localmente nella directory `files`.
```
[vagrant@ansible lab3]$ scp pweb01.example.com:/etc/httpd/conf/httpd.conf files
httpd.conf                                                                                     100%   12KB   3.4MB/s   00:00    
[vagrant@ansible lab3]$ 
```
>4. Modificare il playbook `install_webserver.yml` già presente nella directory `lab3`, in modo da aggiungere dopo l'installazione del pacchetto `httpd` i seguenti task:
>     * inserire un task per la copia del file `files/httpd.conf` nella directory `/etc/httpd/conf/` sul managed host.
>     * inserire un task per l'impostazione dei permessi `644` e ownership `root:root` sul file  `/etc/httpd/conf/httpd.conf`.
>     * inserire un task per la creazione sul managed host del file `/usr/share/httpd/noindex/index.html` con il contenuto `Welcome to my Web Server` (usare il parametro `content`).
>     * modificare il task di avvio del servizio in modo da eseguire il riavvio del servizio.
```yaml
---
# file: install_webserver.yml
# Play 1: Enable apache httpd service
#         Install httpd package, enable and start httpd service

- name: Enable apache httpd service
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Copy httpd.conf
      ansible.builtin.copy:
        src: files/httpd.conf
        dest: /etc/httpd/conf/httpd.conf

    - name: Set permissions on /etc/httpd/conf/httpd.conf
      ansible.builtin.file:
        path: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '644'

    - name: Copy index.html
      ansible.builtin.copy:
        content: 'Welcome to my Web Server'
        dest: /usr/share/httpd/noindex/index.html

    - name: Enable and start httpd service 
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: restarted
```
> 5. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo sulla porta 80.
```
[vagrant@ansible lab3]$ curl http://pweb01.example.com |more
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html lang="en">
...
...
</html>
[vagrant@ansible lab3]$ 
```
> 6. Eseguire una syntax-check sul playbook e se non ci sono errori eseguirlo.
```
[vagrant@ansible lab3]$ ansible-playbook install_webserver.yml

PLAY [Enable apache httpd service] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Copy httpd.conf] **********************************************************************************************************
changed: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ****************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **********************************************************************************************************
changed: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab3]$ 
```
> 7. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http non sia più attivo sulla porta 80 e che invece sia attivo sulla porta 9080.
```
[vagrant@ansible lab3]$ curl http://pweb01.example.com
curl: (7) Failed to connect to pweb01.example.com port 80: Connection refused
[vagrant@ansible lab3]$ curl http://pweb01.example.com:9080
Welcome to my Web Server[vagrant@ansible lab3]$ 
```
> 8. Rieseguire il playbook ed osservare il log di esecuzione
```
[vagrant@ansible lab3]$ ansible-playbook install_webserver.yml

PLAY [Enable apache httpd service] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Copy httpd.conf] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ****************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab3]$ 
```
> 9. Impostare *manualmente* i permessi del file remoto `/etc/httpd/conf/httpd.conf` a `777` (`ssh pweb01.example.com 'sudo chmod 777 /etc/httpd/conf/httpd.conf'`)
```
[vagrant@ansible lab3]$ ssh pweb01.example.com 'sudo chmod 777 /etc/httpd/conf/httpd.conf'
[vagrant@ansible lab3]$ 
```
> 10. Rieseguire il playbook ed osservare il log di esecuzione
```
[vagrant@ansible lab3]$ ansible-playbook install_webserver.yml

PLAY [Enable apache httpd service] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Copy httpd.conf] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ****************************************************************************
changed: [pweb01.example.com]

TASK [Copy index.html] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab3]$ 
```

---
>**Esercizio 2**
>1. Spostarsi nella directory `lab3` e creare la directory `templates` sul control node. 
```
[vagrant@ansible lab3]$ mkdir templates
[vagrant@ansible lab3]$ 
```
>2. Copiare il file `files/httpd.conf` nella nuova directory e aggiungendo il suffisso .j2 `templates/httpd.conf.j2`
```
[vagrant@ansible lab3]$ cp files/httpd.conf templates/httpd.conf.j2
[vagrant@ansible lab3]$ 
```
>3. Modificare il file `templates/httpd.conf.j2` sostituendo la porta `9080` con la variabile ansible `http_port`.
```
Listen {{ http_port }}
```
>4. Modificare il playbook install_webserver.yml già presente nella directory `lab3` in modo da:
>   * sostituire il task di copia del file `files/httpd.conf`, con il task template mantenendo lo stesso path destinatario e prendendo come source il template `templates/httpd.conf.j2`
>   * modificare il task per la creazione del file `/usr/share/httpd/noindex/index.html`, aggiungendo in coda al contenuto il valore dei fact che riportano l'fqdn e l'ip address del managed host.
```yaml
---
# file: install_webserver.yml
# Play 1: Enable apache httpd service
#         Install httpd package, enable and start httpd service

- name: Enable apache httpd service
  hosts: 'pweb01.example.com'
  become: yes

  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Configure & copy httpd.conf
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf

    - name: Set permissions on /etc/httpd/conf/httpd.conf
      ansible.builtin.file:
        path: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '644'

    - name: Copy index.html
      ansible.builtin.copy:
        content: 'Welcome to my Web Server {{ ansible_fqdn }} - {{ ansible_eth0.ipv4.address }}'
        dest: /usr/share/httpd/noindex/index.html

    - name: Enable and start httpd service 
      ansible.builtin.service:
        name: httpd
        enabled: yes
        state: restarted
```
> 5. Eseguire una syntax-check sul playbook e se non ci sono errori eseguirlo.
```
[vagrant@ansible lab3]$ ansible-playbook install_webserver.yml 

PLAY [Enable apache httpd service] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **********************************************************************************************
changed: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ****************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **********************************************************************************************************
changed: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab3]$ 
```
> 6. Eseguire il comando `curl http://pweb01.example.com` sulla porta configurata durante l'esecuzione del playbook.
```
[vagrant@ansible lab3]$ curl http://pweb01.example.com:9080
curl: (7) Failed to connect to pweb01.example.com port 9080: Connection refused
[vagrant@ansible lab3]$ curl http://pweb01.example.com:8080
Welcome to my Web Server pweb01.example.com - 10.0.2.15[vagrant@ansible lab3]$ 
[vagrant@ansible lab3]$ 
```
> 7. Eseguire il playbook passando la variabile http_port=8888 come extra vars.
```
[vagrant@ansible lab3]$ ansible-playbook --extra-vars http_port=8888 install_webserver.yml 

PLAY [Enable apache httpd service] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Install httpd package] ****************************************************************************************************
ok: [pweb01.example.com]

TASK [Configure & copy httpd.conf] **********************************************************************************************
changed: [pweb01.example.com]

TASK [Set permissions on /etc/httpd/conf/httpd.conf] ****************************************************************************
ok: [pweb01.example.com]

TASK [Copy index.html] **********************************************************************************************************
ok: [pweb01.example.com]

TASK [Enable and start httpd service] *******************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **********************************************************************************************************************
pweb01.example.com         : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible lab3]$ 
```
> 8. Eseguire il comando `curl http://pweb01.example.com` sulla nuova porta.
```
[vagrant@ansible lab3]$ curl http://pweb01.example.com:8080
curl: (7) Failed to connect to pweb01.example.com port 8080: Connection refused
[vagrant@ansible lab3]$ curl http://pweb01.example.com:8888
Welcome to my Web Server pweb01.example.com - 10.0.2.15[vagrant@ansible lab3]$ 
[vagrant@ansible lab3]$ 
```

---

