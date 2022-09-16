# Lab2: Variabili e fact
---

**Obiettivo:** Mostrare come usare le variabili e recuperare informazioni dai managed host.

**Ansible variables**

Nel lab1 abbiamo implementato un semplice playbook che installa e attiva il servizio Apache httpd nella configurazione standard. Nella complessa realtà quotidiana sarà invece richiesto che un servizio o comunque una qualunque configurazione debba essere implementata in modo diverso su sistemi diversi o magari su ambienti diversi, si pensi come esemprio alle configurazioni di un sistema/middleware/applicazione in ambient di test e produzione.

Non ci sono dubbi sul fatto che non si possa indirizzare tale esigenza con playbook differenti, ne deriverebbero alti costi di manutenzione e bassa qualità dei playbook stessi.
Le differenze tra i diversi sistemi o ambienti devono invece essere estrapolate dalle configurazioni e gestite in variabili.
In Ansible è possibile creare e gestire le variabili in posti differenti, ad esempio può essere definita nell'inventory, nel play, in file esterni o passata come parametro `--extra-vars` di ansible-playbook.
Se la stessa variabile viene creata in posti differenti, Ansible utilizza una lista per definire la precedenza e quale valore considerare durante l'esecuzione del playbook. Per il lab è sufficiente ricordare la precedenza definita dalla lista seguente, dove l'ultimo della lista è quello a precedenza più alta. Per ulteriori approfondimenti fare riferimento a [Using Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html).

1. inventory file group vars
```yaml
[webservers:vars]
http_port=8080
```
2. inventory file host vars
```yaml
pweb01.example.com http_port=8090
```
3. play vars
```yaml
---
- name: Install apache http
  hosts: webservers:&test
  vars:
    http_port: 7080
```
4. play vars_files
```yaml
---
- name: Install apache http
  hosts: webservers:&test
  vars_files:
    - vars/external.yml
```
5. include_vars
```yaml
    tasks:
      - name: Include all variables defined in vars/external.yml
        ansible.builtin.include_vars:
          file: vars/external.yaml
```
6. set_facts / registered vars
```yaml
- name: Setting host facts using complex arguments
  ansible.builtin.set_fact:
    one_fact: something
```
7. extra vars (for example, -e "user=my_user")(always win precedence)
```
ansible-playbook --extra-vars http_port=7080 install_webserver.yml
```

Si riporta un esempio di file yaml usato per la sola definizione di variabili:

```yaml
---
# file: vars/external.yml
http_port: 8080
```
Il modulo ansible [ansible.builtin.debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html) permette di visualizzare messaggi e valori delle variabili nel log di esecuzione di un playbook, molto utile nella fase di debugging.

>**Esercizio 1**
>Facendo riferimento alla tabella seguente, che riporta i valori da assegnare alla variabile http_port con le modalità indicate:
>1. creare il playbook `lab2/print_http_port.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>    * print della variabile http_port
>    * impostazione del fact http_port
>    * print della variabile http_port
>2. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
>3. impostare nel file `lab2/inventory.ini` la variabile http_port sull'host `pweb01.example.com`
>4. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
>5. modificare il playbook `lab2/print_http_port.yml` impostando la variabile http_port nelle `play vars`
>6. eseguire il playbook e verificare i valori della variabile nel log di esecuzione
>7. eseguire il playbook impostando la variabile http_port come extra vars e verificare i valori della variabile nel log di esecuzione

Tabella a supporto dell'Esercizio 1:
| http_port                 | valore  |
| ------------------------- | ------- |
| inventory file group vars | 8080    |
| inventory file host vars  | 8081    |
| play vars                 | 8082    |
| set_facts                 | 8083    |
| extra vars                | 8084    |

**Ansible facts**

Probabilmente sarà stato notato che nel log di esecuzione di un playbook è sempre presente ed eseguito il `TASK [Gathering Facts]`, che non è dichiarato esplicitamente nel playbook.
Questo task è usato da Ansible per recuperare una serie di variabili contenenti informazioni relative al sistema su cui è in esecuzione il playbook, le variabili relative al sistema remoto sono chiamate facts. I facts permettono la configurazione di un sistema usando le informazioni di altri sistemi.

Il recupero dei fact dai sistemi remoti richiede del tempo, che seppur minimo potrebbe diventare significativo nell'esecuzione del playbook per numerori managed host. Se l'esecuzione di un playbook non necessita dei fact relativi ai sistemi remoti, è possibile disabilitare l'esecuzione del task inserendo la direttiva `gather_facts: false`

```yaml
---
- name: Install apache httpd
  hosts: 'webservers:&test'
  gather_facts: false
  become: yes

  tasks:
    - name: Install http package
      <module>:
      
    - name: Start http service
      <module>:
```
Per ulteriori approfondimenti fare riferimento a [Discovering variables: facts and magic variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)

>**Esercizio 2**
>E' possibile vedere i fact recuperati da un sistema remoto con il modulo [ansible.builtin.setup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html). Eseguire il modulo `ansible.builtin.setup` come un ad-hoc command verso l'host `pweb01.example.com`

Osservando l'output del modulo `ansible.builtin.setup` si osserva che non è un elenco flat di variabili, ma organizzato secondo una struttura di tipo dictionary python. All'interno di un playbook, l'esecuzione esplicita di un task di tipo `ansible.builtin.setup` restituisce la variabile dictionary `ansible_facts` contenente i fact recuperati. Il recupero dei fact impostando `gather_facts: true` valorizza invece direttamente le variabili per poter accedere a specifiche informazioni.

>**Esercizio 3**
>1. creare il playbook `lab2/print_some_facts.yml` per eseguire sul managed host `pweb01.example.com` i seguenti passi:
>    * print dell'hostname
>    * print della distribuzione del sistema operativo
>    * print della versione del sistema operativo
>    * print di tutti gli indirizzi ipv4
>    * print del solo indirizzo ipv4 della scheda eth0
>2. eseguire il playbook e verificare i valori della variabile nel log di esecuzione

Ansible permette anche la gestione e il recupero di fact custom, generati staticamente da altri processi o dinamicamente mediante l'esecuzione di script o eseguibili.
Un file json creato nel path `/etc/ansible/fact.d` e con estensione `.fact` viene automaticamente caricato durante il recupero dei fact di sistema, è possibile gestire i fact anche in path differenti, in tal caso dovranno essere caticati esplicitamente con il modulo `ansible.builtin.setup` e specificando il path in cui cercare i fact.
I file `.fact` con i permessi di esecuzione saranno considerati come fact *dinamici* ed eseguiti, i fact dinamici dovranno avere un output in JSON.

I custom fact recuperati sono valorizzati nel fact `ansible_local`, è possibile passare il parametro `-a filter='ansible_local'` al modulo `ansible.builtin.setup` eseguito come ad-hoc command per visualizzare solo i fact presenti in `ansible_local`.

>**Esercizio 4**
>1. Creare il file `lab2/installed_applications.fact` con il contenuto riportato di seguito
```json
{
    "pr": {
            "name": "Pagamenti e rimborsi",
            "version": "1.1.0"
    },
    "gar": {
            "name": "Gestione accertamenti e regolarizzazione",
            "version": "2.4.5"
    },
    "ic": {
            "name": "Istanze e comunicazioni ",
            "version": "5.2.0"
    }
}
```
>2. Creare il file `lab2/dyncustom.fact` con il contenuto riportato di seguito
```bash
#!/bin/bash

NOW=`date +%Y%m%d-%H.%M.%S`
JSONOUTPUT="{
    \"executiontime\": \"${NOW}\"
}"
echo $JSONOUTPUT
```
>3. Dare i permessi di esecuzione al file `lab2/dyncustom.fact` (`chmod +x dyncustom.fact`)
>4. Copiare i custom fact creati nella directory `/etc/ansible/facts.d` sull'host `pweb01.example.com`
```
[vagrant@ansible lab2]$ ssh pweb01.example.com 'sudo mkdir -p /etc/ansible/facts.d'
[vagrant@ansible lab2]$ scp *.fact pweb01.example.com:/tmp
[vagrant@ansible lab2]$ ssh pweb01.example.com 'sudo mv /tmp/*.fact /etc/ansible/facts.d'
```
>5. Eseguire più volte il modulo `ansible.builtin.setup` come ad-hoc command sul server `pweb01.example.com` e verificarne l'output.

---