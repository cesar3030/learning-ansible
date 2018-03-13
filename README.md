# Ansible Command line et Playbooks
## Introduction
Ce repo contient des exemples d'utilisation d'Ansible en command line et via des playbooks.
Pour executer les commandes et les playbooks, il vous faudra trois machines virtuelles, une servant de maitre (celle sur laquelle Ansible est installé) et deux machines virtuelles esclaves servant de serveur remote auxquels ansible va se connecter via SSH.

Dans notre exemple les trois serveurs sont:
 * Debian Ansible Server: 192.168.56.100
 * Debian 1 (un serveur remote): 192.168.56.101
 * Debian 2 (un serveur remote): 192.168.56.102

## Installation
### VM VirtualBox
1. Créer un "Host-Only Ethernet Adapter" sur lequel toutes les trois serveurs vont se connecter.
2. Dans les propriétés de chaque machine virtuelle, aller dans l'onget "Network" et ajouter un "Adapter" de type "Host-only Adapter". Selectionner à l'aide de la liste déroulante l'adapteur créé lors de l'étape 1 pour le champ "Name".

### Pour tous les serveurs
1. Créer une clé ssh: `ssh-keygen`
2. Ajouter une interface réseau a votre serveur avec une ip differente pour chacun d'entre eux. Pour ce fait, éditer `/etc/network/interfaces` et ajouter l'interface enp0s8: 
``` 
auto enp0s8
iface enp0s8 inet static
address 192.168.56.100
netmask 255.255.255.0
```
Ici nous assignons l'ip `192.168.56.100` au serveur.  
3. Editer le fichier `/etc/hostname` pour donner un nom a votre machine. Dans cet exemple nous avons: 
* ansible-server (ip: 192.168.56.100)
* debian-1 (ip: 192.168.56.101)
* debians-2 (ip: 192.168.56.102)
4. Editer le fichier `etc/hosts` en y ajoutant l'ip du serveur suivi de son hostname. Exemple: `192.168.56.100	ansible-server`.

### Pour le serveur Ansible
Dans cet exemple le serveur Ansible est le serveur Debian ayant l'ip: 192.168.56.100.
1. Installer Ansible: [Instructions ici](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-debian)
2. Ajouter les serveurs remote au fichier `/etc/ansible/hosts`. 
```
[dbservers]
192.168.56.101
192.168.56.102

[debian1]
192.168.56.101

[debian2]
192.168.56.102
```
Ici nous créons trois groupes de serveurs. Nos deux serveurs remote (192.168.56.101 et 192.168.56.102) seront des serveurs de BD. Afin d'executer des commandes sur tous les serveurs de BD à la fois, nous créons un groupe **dbservers** avec l'ip de nos deux serveurs. Deux groupes **debians1** et **debian2** permettenent d'acceder à chaque serveur remote individuellement.
3. Partager la cle publique SSH du serveur Ansible avec les remote: `ssh-copy-id -i ~/.ssh/id_rsa <votre_user>@192.168.556.101` et `ssh-copy-id -i ~/.ssh/id_rsa <votre_user>@192.168.556.102`

Votre serveur Ansible est désormait pret à etre utilisé!!

## Ansible: Command Line
Il est possible d'utiliser Ansible pour effectuer de simples manipulations sur des serveurs remote via le command line. La syntaxe de base est la suivante: `ansible <host_groupe> -m <nom_du_module> -a <arguments_du_module>`.  
Voici une liste non exaustives des opérations possibles de faire:
* Exécuter la commande shell `cat /etc/hostname` sur tous les serveurs BD (`dbservers`)  afin de nous retourner le hostname de chaque serveur: `ansible dbservers -m shell -a 'cat /etc/hostname'` 
