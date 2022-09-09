#Ansible Labs
---

## Predisposizione ambiente Lab
Gli esercizi possono essere eseguiti in un ambiente lab virtuale creato automaticamente sulla propria postazione di lavoro.

I tool necessari e che dovranno essere preventivamente installati sono:
* [VirtualBox](https://www.virtualbox.org)
* [Vagrant](https://www.vagrantup.com)


### Installazione VirtualBox
Scaricare il pacchetto di installazione specifico per la propria piattaforma dal sito ufficiale di VirtualBox https://www.virtualbox.org/wiki/Downloads .
Eseguire una installazione standard, lasciando tutte le impostazioni di default.

>Nel caso in cui sia installato Microsoft Hyper-V fare riferimento all'articolo [Le applicazioni di virtualizzazione non funzionano insieme a Hyper-V, Device Guard e Credential Guard](https://docs.microsoft.com/it-it/troubleshoot/windows-client/application-management/virtualization-apps-not-work-with-hyper-v)

### Installazione Vagrant
Scaricare il pacchetto di installazione specifico per la propria piattaforma dal sito uffiziale di Vagrant https://www.vagrantup.com/downloads.
Eseguire una installazione standard, lasciando le impostazioni di default. 
Al termine dell'installazione sarà richiesto un riavvio del computer.

### Verifica ambiente lab
Le macchine virtuali sono definite e configurate tramite un file `Vagrantfile`.
Il comando seguente inizializza un Vagrantfile con impostazioni di default utilizzando l'immagine di sistema operativo (box) specificata.

```
C:\projects\vagrant-test>vagrant init centos/stream8
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Il comando `vagrant up` esegue la creazione e attivazione dell'ambiente virtuale, in particolare provvede a:
* Scaricare/aggiornare il box specificato nel Vagrantfile
* Creare la VM nell'ambiente di virtualizzazione configurato (VirtualBox)
* Attivare e configurare la VM


```
C:\projects\vagrant-test>vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'centos/stream8' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'centos/stream8'
    default: URL: https://vagrantcloud.com/centos/stream8
==> default: Adding box 'centos/stream8' (v20210210.0) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/centos/boxes/stream8/versions/20210210.0/providers/virtualbox.box
Download redirected to host: cloud.centos.org
    default:
    default: Calculating and comparing box checksum...
==> default: Successfully added box 'centos/stream8' (v20210210.0) for 'virtualbox'!
==> default: Importing base box 'centos/stream8'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'centos/stream8' version '20210210.0' is up to date...
==> default: Setting the name of the VM: vagrant-test_default_1662591043284_81647
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default:
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
==> default: Rsyncing folder: /cygdrive/c/projects/vagrant-test/ => /vagrant
```

In qualunque momento è possibile verificare lo stato delle macchine definite nel Vagrantfile, tramite il comando `vagrant status`

```
C:\projects\vagrant-test>vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

Il comando `vagrant ssh` esegue la connessione in ssh con la macchina virtuale creata. Nel caso in cui nel Vagrantfile siano definite più macchine si dovrà aggiungere il nome della macchina, es. `vagrant ssh machine1`.
Collegarsi alla macchina di default creata ed eseguire il comando di update del sistema per verificare che la configurazione di rete della vm permetta l'accesso a internet.

```
C:\projects\vagrant-test>vagrant ssh
[vagrant@localhost ~]$
[vagrant@localhost ~]$ sudo yum update
```

L'intero ambiente virtuale può essere rimosso con il comando `vagrant destroy`, che provvederà a spegnere le macchine e a cancellare le VM create nel sistema di virtualizzazione utilizzato.

```
C:\projects\vagrant-test>vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```


