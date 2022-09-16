# Lab1: Ansible inventory e primo playbook
---

**Obiettivo:** Mostrare la modalità di selezione dei managed host dall'inventory e scrivere il primo playbook Ansible.

**Ansible inventory**

Nel lab0 abbiamo usato il modulo `ping` come un ad-hoc command, verificare la sintassi per l'esecuzione dei comandi ad-hoc con `ansible --help`. Tra i parametri esiste il parametro per specificare il file di inventory, in cui è configurata l'elenco dei managed host.

```
optional arguments:
...
  -i INVENTORY, --inventory INVENTORY, --inventory-file INVENTORY
                        specify inventory host path or comma separated host list. --inventory-file is deprecated
...
```

Nella verifica della connessione con il server `pweb01.example.com` non abbiamo specificato l'inventory perchè è stato preventivamente impostato nel file di configurazione `ansible.cfg`, presente nella directory in cui abbiamo eseguito ansible. Aprire il file `ansible.cfg` e `inventory.ini` da VSCode per vederne il contenuto.

Un parametro che invece abbiamo specificato, e che è obbligatorio, è l'`host pattern` che deve essere specificato come ultimo parametro.

```
positional arguments:
  pattern               host pattern
```

L'host pattern seleziona gli host su cui eseguire l'ad-hoc command o il playbook tra quelli configurati nell'inventory.
Il parametro --list-hosts riporta l'elenco degli host risolti dall'host pattern specificato, per selezionare tutti gli host presenti nell'inventory si usa il pattern `all`.

```
[vagrant@ansible lab0]$ cd ../lab1
[vagrant@ansible lab1]$ 
[vagrant@ansible lab1]$ ansible --list-hosts all
  hosts (7):
    lb.example.com
    tweb01.example.com
    pweb01.example.com
    pweb02.example.com
    tdb01.example.com
    pdb01.example.com
    pdb02.example.com
[vagrant@ansible lab1]$ 
```

>**Esercizio 1**
>Facendo riferimento al file `labs/lab1/inventory.ini` e alla documentazione [Patterns: targeting hosts and groups](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html), identificare e provare con il parametro --list-hosts gli host che soddisfano i seguenti criteri:
>1. Tutti i web server
>2. Tutti i server di produzione
>3. Tutti i web server di produzione
>4. Tutti i server che non sono di test
>5. Tutti i server che non sono di test e non sono db server

**Ansible playbook**

Il playbook è un file scritto in yaml che contiene uno o più play, ogni play definisce
* i task che devono essere eseguiti sequenzialmente per portare i managed host ad una determinata configurazione
* i managed host selezionati dall'inventory su cui eseguire i task

Un esempio di playbook è

```yaml
---
- name: Install apache httpd
  hosts: 'webservers:&test'

  tasks:
    - name: Install http package
      <module>:
      
    - name: Start http service
      <module>:

- name: Install mysql
  hosts: 'dbservers:&test'

  tasks:
    - name: Install mysql package
      <module>:

    - name: Create database
      <module>:
```

I task sono eseguiti sui managed host con l'utenza specificata per la connessione remota agli host, che può essere specificata nel seguente ordine di priorità:

* da linea comando tramite il parametro  ` -u REMOTE_USER` o ` --user REMOTE_USER`
* nel file ansible.cfg `remote_user = REMOTE_USER`
* l'utenza con cui viene eseguito l'ad-hoc command o il playbook

Alcuni task hanno però bisogno di una utenza diversa per poter essere eseguiti, ad esempio serve l'utenza root per poter installare dei pacchetti con il package manager o l'utenza con cui è in esecuzione un servizio per modificare i file usati dal servizio.
Ansible utilizza i meccanismi di privilege escalation esistenti sul sistemi target (es. sudo e su) per eseguire task e play con i privilegi di root o di altre utenze. 
La direttiva che controlla la escalation è `become`, quella per indicare l'utenza è `become_user` (root se non specificato). Possono essere configurate a livello di play, eseguendo tutti i task definiti nel play con quella utenza, o a livello di singolo task. Per approfondimenti [Understanding privilege escalation: become](https://docs.ansible.com/ansible/latest/user_guide/become.html).

```yaml
---
- name: Install apache httpd
  hosts: 'webservers:&test'
  become: yes

  tasks:
    - name: Install http package
      <module>:
      
    - name: Start http service
      <module>:
```

Un playbook viene eseguito con il comando `ansible-playbook playbook.yml`, il comando prevede un parametro `--syntax-check` per verificare la correttezza sintattica del playbook.

>**Esercizio 2**
>Facendo riferimento alla documentazione dei moduli [ansible.builtin.yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html) e [ansible.builtin.service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html):
> 1. creare il playbook `lab1/install_webserver.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>     * installazione del pacchetto **httpd**
>     * abilitazione del servizio **httpd** al boot e avvio del servizio
> 2. verificare che il playbook sia sintatticamente corretto
> 3. eseguire il playbook
> 4. eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo
> 5. rieseguire il playbook
> 6. eseguire il comando `ssh pweb01 'sudo systemctl stop httpd'` e rieseguire il playbook
> 7. osservare l'output delle tre esecuzioni del playbook e commentare
>
> *Nota:* La documentazione dei moduli builtin, e di tutti quelli eventualmente installati sul controller node, può essere visionata con il comando `ansible-doc <modulo>` 

---