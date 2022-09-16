# Ansible Labs
---

Questo repository contiene una serie di esercitazioni su Ansible.
L'ambiente in cui eseguire gli esercizi viene predisposto sulla postazione personale utilizzando VirtualBox, un prodotto di virtualizzazione disponibile gratuitamente. Tutte le istruzioni per l'installazione e configurazione dei tool necessari sono riportati nella sezione specifica.

* [Installazione Lab tools](setup/README.md)
* [Predisposizione ambiente ](#predisposizione-ambiente)
* [Labs](labs/README.md)

## Predisposizione ambiente
>Questa sezione ha come prerequisito il completamento della sezione [Installazione Lab tools](setup/README.md).

Scaricare sulla propria postazione questo repository effettuando una `git clone` (è richiesto che sia installato il tool git):

```
C:\projects>git clone https://github.com/esalierno/ansible-labs
```
In alternativa è possibile utilizzare la funzione `Download ZIP` disponibile in ogni repository GitHub:

![Download ZIP](/images/github-download-zip.png)

Avviare Visual Studio Code ed aprire il folder (File->Open Folder...) dove è stato clonato o scompattato il repository (es. C:\projects\ansible-labs). Trustare il folder come indicato di seguito:

![VSCode Open Project](/images/vscode-prj-1.png)

Aprire il file `ssh.config`, modificare la prima parte del path della proprietà IdentityFile in modo che corrisponda al path in cui è stato clonato/scompattato il repository e salvere le modifiche (Ctrl+S o File->Save).

![VSCode Change ssh.config](/images/vscode-prj-2.png)

Aprire un terminale (Terminal->New Terminal) ed eseguire il comando `vagrant up ansible pweb01` per avviare e configurare le prime VM necessarie all'esecuzione dei lab. Nel nostro ambiente la VM `ansible` è il controller node su cui sarà installato ed eseguito ansible, mentre la VM `pweb01` è un managed node.

![VSCode Vagrant up](/images/vscode-prj-3.png)

```
PS C:\projects\ansible-labs> vagrant up ansible pweb01
Bringing machine 'ansible' up with 'virtualbox' provider...
Bringing machine 'pweb01' up with 'virtualbox' provider...
==> ansible: Cloning VM...
==> ansible: Matching MAC address for NAT networking...
==> ansible: Checking if box 'centos/stream8' version '20210210.0' is up to date...
==> ansible: Setting the name of the VM: ansible-labs_ansible_1663228381567_74800
...
...
```

In ogni momento possiamo verificare lo stato delle VM del nostro ambiente con il comando `vagrant status`.

```
PS C:\projects\ansible-labs> vagrant status
Current machine states:

ansible                   running (virtualbox)
lb                        not created (virtualbox)
pweb01                    running (virtualbox)
pweb02                    not created (virtualbox)
pdb01                     not created (virtualbox)
pdb02                     not created (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Nella sessione di VSCode attuale, aprire il Remote Explorer ed impostare la configurazione SSH usando un file di configurazione custom.

![VSCode Configure Remote SSH](/images/vscode-prj-4.png)

Nella finestra di configurazione impostare il path del file di configurazione ssh presente nel repository (es. C:\projects\ansible-labs\ssh.config), VSCode legge immediatamente la nuova configurazione e visualizza l'host **ansible** tra gli SSH TARGETS. E' ora possibile collegarsi all'host ansible aprendo una nuova sessione VSCode clickando sull'icona alla destra dell'host. 

![VSCode Open remote ssh target](/images/vscode-prj-5.png)

Selezionare Linux come piattaforma remota a cui si ci sta collegando, in modo che VSCode possa scaricare sull'host remoto le estensioni compatibili con l'host remoto.

![VSCode Open remote ssh target](/images/vscode-prj-6.png)

Aprire la sezione delle estensioni di VSCode e installare sull'host remoto l'estensione Ansible, precedentemente installata nella sessione locale di VSCode.

![VSCode Install Ansibile extension on remote ssh target](/images/vscode-prj-7.png)

Aprire (File->Open Folder...) il folder di default (/home/vagrant), trustare il folder e aprire un terminale (Terminal->New Terminal).

![VSCode Open remote ssh target](/images/vscode-prj-8.png)


Siamo pronti per iniziare i [Labs](labs/README.md).