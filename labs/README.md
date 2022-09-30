# Ansible Labs
---

I lab presenti in questa sezione hanno l'obiettivo di mostrare ed introdurre le funzionalità messe a disposizione da Ansible a supporto della configurazione dei sistemi e dell'automazione più in generale.

Ogni lab ha come prerequisito l'esecuzione dei lab precedenti.

>Qualora si abbia la necessità di interrompere le esercitazioni è possibile salvare in qualunque momento e in modo persistente lo stato delle VM con il comando `vagrant suspend`, per la riattivazione delle VM usare il comando `vagrant resume`.

```
PS C:\projects\ansible-labs> vagrant suspend
==> ansible: Saving VM state and suspending execution...
==> lb: VM not created. Moving on...
==> pweb01: Saving VM state and suspending execution...
==> pweb02: VM not created. Moving on...
==> pdb01: VM not created. Moving on...
==> pdb02: VM not created. Moving on...
PS C:\projects\ansible-labs> 
```
## Indice Labs

* [Lab0: Installazione ansible sul control node](lab0/README.md)
  
  Predisposizione del control node e i managed host per l'esecuzione dei playbook Ansible.

* [Lab1: Ansible inventory e primo playbook](lab1/README.md)
  
  Overview sulla modalità di selezione dei managed host dall'inventory e implementazione del primo playbook Ansible.

* [Lab2: Variabili e fact](lab2/README.md)
  
  Overview sulle variabili e sulle capacità di Ansible per recuperare informazioni dai managed host.

* [Lab3: Copia file e modifica file di configurazione](lab3/README.md)
  
  Overview sui moduli per la copia di file sui managed host e per la configurazione dinamica di file di configurazione.

* [Lab4: Esecuzione condizionata dei task](lab4/README.md)

  Overview sulle modalità per vincolare l'esecuzione dei task al valore di variabili/fact e all'esito/stato di task precedenti.


   