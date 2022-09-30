# Lab4: Esecuzione condizionata dei task
---

**Obiettivo:** Mostrare come poter eseguire dei task esclusivamente in seguito all'applicazione di change da altri task. 

**Recupero output di un task ed esecuzione condizionale**

Negli esercizi del lab3 è stato mostrato come modificare la configurazione di un'applicazione o di un servizio, usando file di configurazione statici o template popolati dinamicamente tramite jinja2 e le variabili. Affinchè la nuova configurazione sia applicata è quasi sempre richiesto il riavvio del servizio (alcuni servizi prevedono una reload della configurazione), motivo per cui negli esercizi è stato necessario modificare lo `state` del task relativo al servizio httpd da `started` a `restarted`.
Eseguendo più volte il playbook senza modificare la configurazione del servizio httpd, si nota che tutti i task di installazione e configurazione non comportano `change` al managed host, mentre il riavvio del servizio esegue sempre una modifica dello stato del managed host e quindi viene sempre sentito come `changed`.

```
...
...
TASK [Enable and restart httpd service] ***********************************************************************************************************************************************************
changed: [pweb01.example.com]

PLAY RECAP **************************************************************************************************************************************************************************************
pweb01.example.com         : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Il riavvio del servizio è task non idempotente, affinchè il playbook resti idempotente è necessario che il riavvio del servizio sia eseguito solo quando effettivamente necessario e quindi solo quando la configurazione viene modificata.
E' quindi necessario catturare l'attributo "changed" dei task responsabili della configurazione del servizio per poter poi decidere se eseguire o meno il task del riavvio del servizio stesso.
Ansible permette la creazione di variabili popolate automaticamente con il risultato dell'esecuzione di un task impostando il parametro [register](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#registering-variables). Alla variabile viene assegnato un valore di tipo dictionary che include, oltre gli attributi specifici del task eseguito, gli attributi `changed` e `failed`.

```json
{
    "copy_output": {
        "changed": false,
        "failed": false,
...
...
    }
}
```

Prima di usare la variabile registrata è opportuno conoscere gli attributi riportati nel dictionary, in modo da poter referenziare correttamente quello di interesse. E' quindi utile inserire un task con il modulo [ansible.builtin.debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html) subito dopo il task in cui viene registrata una variabile di output, che permette di loggare la variabile nel log di esecuzione del playbook:

```yaml
    - name: Debug copy index.html
      debug:
        var: copy_output
        verbosity: 0
```
La verbosity definisce quando eseguire il task debug, con verbosity=0 viene sempre eseguito, con verbosity=1 solo se il playbook è eseguito con il parametro "-v". (verbosity=2 parametro -vv... ecc.).

In un playbook è possibile condizionare l'esecuzione di un task al valore di una variabile o di un fact, funzione necessaria quando è richiesta l'implementazione di playbook eseguibili in scenari differenti. Ad esempio, nel playbook implementato durante gli esercizi è stato usato il modulo `ansible.builtin.yum` per l'installazione del pacchetto httpd, assumendo che il playbook sia eseguito su managed host della famiglia "RedHat", non potrebbe quindi essere eseguito su host della famiglia "Debian" che usano un differente package manager (apt).

L'esecuzione condizionata è utile anche in quei casi in cui siamo costretti ad usare moduli non idempotenti, un esempio su tutti è il modulo [ansible.builtin.command](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html) che permette l'esecuzione sul managed host di comandi o script disponibili sul sistema target. Il modulo `ansible.builtin.command` non è in grado di capire se il comando eseguito abbia realmente apportato un change al host, pertanto di default assume che lo stato dell'host sia cambiato (changed: true).
Il task prevede il parametro `changed_when` ([Error handling in playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)) per impostare il valore restituito come changed, è possibile impostare una condizione calcolata analizzando return code o stdout del comando, o piuttosto impostarlo staticamente nel caso in cui è certo che il comando eseguito non comporta change al managed host.

```yaml
    - name: ls su managed host
      ansible.builtin.command:
        cmd: 'echo $HOME'
      changed_when: false
```

Nel nostro caso abbiamo sempre l'esigenza di eseguire il riavvio del servizio solo quando la configurazione del servizio è cambiata. Andremo quindi a modificare il task di riavvio per condizionarlo allo stato dei task di configurazione.
Aapprofondimenti sull'esecuzione condizionale: [Conditionals](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html).

Prima di procedere con l'esecizio è necessario copiare il risultato del lab3:

```
[vagrant@ansible ~]$ cd labs/
[vagrant@ansible labs]$ cp -r lab3 lab4
[vagrant@ansible labs]$ cd lab4
[vagrant@ansible lab4]$ 
```

L'obiettivo del prossimo esercizio è modificare il playbook `install_webserver.yml` affinchè il task di riavvio sia eseguito solo quando almeno un task cambi la configurazione dell'http server. Abilitare inoltre l'esecuzione del playbook a piattaforme della famiglia Debian, verificando il fact `ansible_os_family` in modo da condizionare l'esecuzione del task `ansible.builtin.yum` solo a piattaforme "RedHat" e aggiungendo il task [ansible.builtin.apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html) per le piattaforme "Debian".

>***Tip:***
> Per verificare il valore del fact richiesto usare il modulo setup come ad-hoc command: `ansible pweb01.example.com -m setup -a filter='ansible_os_family'`

>***Tip:*** A causa delle policy SELinux attive, non è possibile associare al servizio http una port diversa dalla porta 80.
>Per superare il problema senza andare a modificare le policy SELinux, prima di eseguire il playbook indicato nell'esercizio disabilitare temporaneamente SELinux con il comando:
>* `ssh pweb01.example.com 'sudo setenforce 0'`

>**Esercizio 1**
>1. Modificare il playbook in modo da:
>     * Modificare il task YUM in modo che sia eseguito solo su piattaforme RedHat.
>     * Aggiungere un task per l'installazione del package https su piattaforme Debian, in modo che sia eseguito solo su piattaforme Debian.
>     * Aggiungere ai task che impattano sulla configurazione dell'http server la registrazione di una variabile di output.
>     * Subito dopo ogni task in cui è configurata la registrazione di una variabile di output, aggiungere un task per loggare, quando il playbook è eseguito in verbose, il valore della variabile registrata.
>2. Eseguire il playbook con il parametro "-v" ed osservare il log di esecuzione.
>3. Modificare il task di riavvio del servizio in modo che sia eseguito solo quando almeno uno dei task modificati al punto 1 risulti changed.
>4. Eseguire il playbook e verificare che il servizio non sia stato riavviato.
>5. Eseguire il comando `curl http://pweb01.example.com:8080` per verificare che il server http sia attivo sulla porta 8080. (configurata nell'inventory)
>6. Eseguire il playbook passando la variabile http_port=8888 come extra vars e dal log di esecuzione verificare che il servizio sia stato riavviato.
>7. Eseguire il comando `curl http://pweb01.example.com` per verificare che il server http sia attivo sulla porta 8888.
>


**Handler**

Abbiamo visto come sia possibile vincolare l'esecuzione di un task al verificarsi di una condizione e in particolare abbiamo sfruttato questa funzione per eseguire il task di riavvio del servizio solo se durante l'esecuzione del playbook sono presenti change che lo richiedono.
In realtà per gestire questo tipo di esigenze Ansible prevede dei task particolari chiamati `handlers` (rif. [Handlers: running operations on change](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)).
I task handler sono identici ai task normali, ma invece di essere definiti nella sezione `tasks:` sono definiti in una sezione chiamata appunto `handlers:` in coda al playbook.
Un task può specificare l'esecuzione di un handler, qualora la sua esecuzione risulti changed, utilizzando il parametro `notify` seguito dal nome del task handler (o da una lista nel caso un cui debba notificarne più di uno).

Nel prossimo esercizio cambieremo la gestione del task per il riavvio dell'http server, gestendolo come handler.

>**Esercizio 2**
>1. Modificare il playbook in modo da:
>     * Spostare il task di riavvio del servizio nella sezione handlers ed eliminare le condizioni `when:`.
>     * Rimuovere la registrazione delle variabili di output e i task ansible.builtin.debug.
>     * Aggiungere ai task che richiedono il riavvio del servizio, la notifica all'handler.
>2. Eseguire il playbook e verificare che il servizio sia stato riavviato.
>3. Eseguire nuovamente il playbook e verificare che il servizio non sia stato riavviato.

---