# Lab3: Copia file e modifica file di configurazione
---

**Obiettivo:** Mostrare come copiare file sui managed host e configurare dinamicamente i file di configurazione.

**Configurazione e copia file**

Nella configurazione dei managed host è frequente la necessità di eseguire operazioni con i file:
* creare o rimuovere directory
* creare o rimuovere file
* impostare properties specifiche sui file (ownership, access mode)

Ipotizzando la configurazione di un Apache httpd, nel Lab 1 è stato implementato un playbook per l'installazione del server http con la configurazione di base. Nella realtà i sistemi sono predisposti con configurazioni ben definite, dobbiamo pertanto modificare il playbook in modo da poter caricare sul managed host la nostra configurazione.

I moduli ansible utilizzati nel prossimo esercizio sono:
* [ansible.builtin.file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
  Permette di creare/rimuovere/configurare file, directory e link simbolici (la creazione di file vuoti).
* [ansible.builtin.copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
  Permette la creazione di file con un contenuto impostato e la copia di file dal controller node al managed host.

L'obiettivo del prossimo esercizio è completare l'implementazione del playbook creato nel Lab 1 in modo che possa copiare una `httpd.conf` e un `index.html` customizzato.
Nell'esercizio andremo a recuperare l'httpd.conf installato con il prodotto e cambieremo la configurazione sul control node.


>***Tip:*** A causa delle policy SELinux attive, non è possibile associare al servizio http una port diversa dalla porta 80.
>Per superare il problema senza andare a modificare le policy SELinux, prima di eseguire il playbook indicato nell'esercizio disabilitare temporaneamente SELinux con il comando:
>* `ssh pweb01.example.com 'sudo setenforce 0'`

>**Esercizio 1**
>1. Spostarsi nella directory `lab3` e creare la directory `files` sul control node.
>2. Usando il comando Linux `scp <host>:<path file remoto> <path locale>`, recuperare dal managed host `pweb01.example.com` il file `/etc/httpd/conf/httpd.conf` e copiarlo localmente nella directory `files`.
>3. Modificare il file `files/httpd.conf` cambiando la porta in ascolto da `Listen 80` a `Listen 9080`.
>4. Modificare il playbook `install_webserver.yml` già presente nella directory `lab3`, in modo da aggiungere dopo l'installazione del pacchetto `httpd` i seguenti task:
>     * inserire un task per la copia del file `files/httpd.conf` nella directory `/etc/httpd/conf/` sul managed host.
>     * inserire un task per l'impostazione dei permessi `644` e ownership `root:root` sul file  `/etc/httpd/conf/httpd.conf`.
>     * inserire un task per la creazione sul managed host del file `/usr/share/httpd/noindex/index.html` con il contenuto `Welcome to my Web Server` (usare il parametro `content`).
>     * modificare il task di avvio del servizio in modo da eseguire il riavvio del servizio.
> 5. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo sulla porta 80.
> 6. Eseguire una syntax-check sul playbook e se non ci sono errori eseguirlo.
> 7. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http non sia più attivo sulla porta 80 e che invece sia attivo sulla porta 9080.
> 8. Rieseguire il playbook ed osservare il log di esecuzione
> 9. Impostare *manualmente* i permessi del file remoto `/etc/httpd/conf/httpd.conf` a `777` (`ssh pweb01.example.com 'sudo chmod 777 /etc/httpd/conf/httpd.conf'`)
> 10. Rieseguire il playbook ed osservare il log di esecuzione
>



**Configurazione dinamica dei file**

Come già discusso nel lab sulle variabili e sui fact, è frequente la necessità di configurare dei file in modo diverso su host differenti.
Ansible prevede il modulo [ansible.builtin.template](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html) che permette la sostituzione delle variabili inserite in un template con i valori assunti dalle variabili al tempo dell'esecuzione del playbook e la successiva copia del file sul managed host.

La flessibilità dei template in Ansible è data dal linguaggio di templating Jinja2, che rende possibile implementare template anche molto complessi. Jinja2 è utilizzato anche nei `filter` builtin, che vedremo in seguito.

In questo esercizio useremo Jinja2 in modo molto basilare, per la valorizzazione della porta in ascolto della configurazione del server web e per aggiungere il valore dell'FQDN e IP address nel messaggio di Welcome. 
E' sufficiente sapere che all'interno di un template una variabile ansible deve essere referenziata con la sintassi `{{ variabile ansible }}`.

>**Esercizio 2**
>1. Spostarsi nella directory `lab3` e creare la directory `templates` sul control node. 
>2. Copiare il file `files/httpd.conf` nella nuova directory e aggiungendo il suffisso .j2 `templates/httpd.conf.j2`
>3. Modificare il file `templates/httpd.conf.j2` sostituendo la porta `9080` con la variabile ansible `http_port` (configurata nell'inventory).
>4. Modificare il playbook install_webserver.yml già presente nella directory `lab3` in modo da:
>   * sostituire il task di copia del file `files/httpd.conf`, con il task template mantenendo lo stesso path destinatario e prendendo come source il template `templates/httpd.conf.j2`
>   * modificare il task per la creazione del file `/usr/share/httpd/noindex/index.html`, aggiungendo in coda al contenuto il valore dei fact che riportano l'fqdn e l'ip address del managed host.
> 5. Eseguire una syntax-check sul playbook e se non ci sono errori eseguirlo.
> 6. Eseguire il comando `curl http://pweb01.example.com` sulla porta configurata durante l'esecuzione del playbook.
> 7. Eseguire il playbook passando la variabile http_port=8888 come extra vars.
> 8. Eseguire il comando `curl http://pweb01.example.com` sulla nuova porta.





---