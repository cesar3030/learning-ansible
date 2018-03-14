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

### Executer une commande Shell
Exécuter la commande shell `cat /etc/hostname` sur tous les serveurs BD (`dbservers`)  afin de retourner le hostname de chaque serveur: `ansible dbservers -m shell -a 'cat /etc/hostname'`  
Réponse: 
```
192.168.56.101 | SUCCESS | rc=0 >>
debian1

192.168.56.102 | SUCCESS | rc=0 >>
debian2
```

### Copier un fichier du serveur Ansible aux serveurs remote
Cette commande copie le fichier `text.txt` situé dans le répertoire `/tmp` du serveur Ansible dans le dossier `/tmp` des serveurs remote.
`ansible dbservers -m copy -a "src=/tmp/test.txt dest=/tmp"`  
Réponse:
```
192.168.56.101 | SUCCESS => {
	"changed": true,
	"dest": "/tmp/test.txt",
	"checksum": "fef4ew6g41rgwg46wrg6r4g65wr4g65rwg"
	"gid": 1000,
	"group": "cesar",
	"mode": "0644",
	"owner": "cesar",
	"size": 0,
	"src": "/home/cesar/.ansible/tmp/ansible-tmp-1520948277.38-486454654561/source",
	"state": "file",
	"uid": 1000
}

192.168.56.102 | SUCCESS => {
	"changed": true,
	"dest": "/tmp/test.txt",
	"checksum": "fef4ew6g41rgwg46wrg6r4g65wr4g65rwg"
	"gid": 1000,
	"group": "cesar",
	"mode": "0644",
	"owner": "cesar",
	"size": 0,
	"src": "/home/cesar/.ansible/tmp/ansible-tmp-1520948277.38-486454654561/source",
	"state": "file",
	"uid": 1000
}
```
### Changer les permissions d'un fichier
Cette commande va changer la permission sur les serveurs remote du fichier au path spécifié: `ansible dbservers -m file -a "dest=/tmp/test.txt mode=777"`  
Réponse:
```
192.168.56.101 | SUCCESS => {
	"changed": true,
	"gid": 1000,
	"group": "cesar",
	"mode": "0777",
	"owner": "cesar",
	"path": "/tmp/test.txt"
	"size": 0,
	"state": "file",
	"uid": 1000
}

192.168.56.102 | SUCCESS => {
	"changed": true,
	"gid": 1000,
	"group": "cesar",
	"mode": "0777",
	"owner": "cesar",
	"path": "/tmp/test.txt"
	"size": 0,
	"state": "file",
	"uid": 1000
}
```
Note: On peut aussi changer le owner, le groupe du fichier via cette commande

### Créer un dossier
Cette commande créé la structure de dossiers `dir_from/ansible/` sur les serveurs remote dans le dossier `/tmp`. De plus, ces dossiers auront le mode, l'owner et le groupe.
`ansible dbserver -m file -a "dest=/tmp/dir_from/ansible mode=755 owner=cesar group=cesar state=directory"`  
Réponse
```
192.168.56.101 | SUCCESS => {
	"changed": true,
	"gid": 1000,
	"group": "cesar",
	"mode": "0755",
	"owner": "cesar",
	"path": "/tmp/dir_from/ansible"
	"size": 4096,
	"state": "directory",
	"uid": 1000
}

192.168.56.102 | SUCCESS => {
	"changed": true,
	"gid": 1000,
	"group": "cesar",
	"mode": "0755",
	"owner": "cesar",
	"path": "/tmp/dir_from/ansible"
	"size": 4096,
	"state": "directory",
	"uid": 1000
}

```

### Supprimer un dossier
Cette commande supprime le dossier spécifié en argument ainsi que ses sous dossiers.
`ansible dbservers -m file -a "dest=/tmp/dir_from state=absent"`  
Réponse
```
192.168.56.101 | SUCCESS => {
	"changed": true,
	"path": "/tmp/dir_from/ansible"
	"state": "absent"
}

192.168.56.102 | SUCCESS => {
	"changed": true,
	"path": "/tmp/dir_from/ansible"
	"state": "absent"
}
```

### S'assurer qu'un packet est installé
Cette commande vérifie que le packet sudo est bien présent sur le serveur remote
`ansible dbservers apt -a "name=sudo state=installed"`  
Réponse
```
192.168.56.101 | SUCCESS => {
	"cache_update_time": 1520017448,
	"cache_updated": false,
	"changed": false
}

192.168.56.102 | SUCCESS => {
	"cache_update_time": 1520017449,
	"cache_updated": false,
	"changed": false
}
```  
Note: On peut aussi installer ou désinstaller des packets via ce module en utilisant l'argument `state=present` ou `state=absent`

## Ansible: Playbook

Ansible Playbook sont des fichiers YAML permettant d'executer une succession de taches sur un ou plusieurs groupes de serveurs.
Exemple de playbook avec une tache consernant les hosts `dbservers`.
```yaml
---
- hosts: dbservers          # <== nom du groupe de serveurs sur lesquels opérer
  tasks:					# <== liste des taches à executer sur les serveurs du groupe
    - name: Create a new file in /tmp 	# <== Nom de la tache
      file:								# <== Nom du module utilisé
        path: /tmp/test_no_var.txt 		# <== Argument path du module file et sa valeur 
        state: touch					# <== Argument state du module file et sa valeur 
```

### Créer un fichier
Pour créer un fichier, utiliser le module **file** et procurer le *path* et le *state=touch*.
```yaml
---
- hosts: dbservers
  tasks:
    - name: Create a new file in /tmp
      file:
        path: /tmp/test_no_var.txt
        state: touch
```
### Supprimer un fichier
Pour supprimer un fichier, utiliser le module **file** et procurer le *path* et le *state=absent*.
```yaml
---
- hosts: dbservers
  tasks:
    - name: Create a new file in /tmp
      file:
        path: /tmp/test_no_var.txt
        state: absent
```
### Créer un fichier avec un nom contenu dans une variable
Ce playbook introduit la notion de variables. Il est possible de définir des variables dans un playbook pour ensuite les utiliser dans une tache ou dans un template. Ce playbook va créer le fichier `/tmp/file_from_ansible_variable_name.txt`.
```yaml
---
- hosts: dbservers
  vars : 											# <== Liste de variables
    file_name: file_from_ansible_variable_name.txt 
  tasks:
    - name: Create a new file in /tmp
      file:
        path: /tmp/{{ file_name }}
        state: touch
```
### Créer un fichier depuis un template
Ce playbook initialise des variables et créé un fichier `index.html` dans le dossier `/tmp` des serveurs basé sur le template situé dans le dossier `templates/`. Le template est une page HTML dans laquelle on va remplacer `{{ page_title }}` et `{{ page_body }}` par les valeurs des variables initialisées dans la partie **vars** du playbook.
```yaml
---
- hosts: dbservers
  vars :
    page_title: My first template use!
    page_body: Hello World!
    filename: index.html
  tasks:
    - name: Create a new file in /tmp
      template:
        src: templates/index.html.j2  # Va chercher le template dans le dossier `templates/`
        dest: /tmp/{{filename}}
```
Template: Page HTML `template/index.html.j2`:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>{{ page_title }}</title>
  </head>
  <body>
    {{ page_body }}
  </body>
</html>
```
### Copier un fichier d'un serveur distant à un autre serveur distant
### Créer un fichier sur un serveur remote et le copier sur un autre serveur distant
### Un serveur distant ping un autre serveur distant

### Ansible roles
#### Server role
#### MySQL role
### Dump une base de données
### Dump, Copie le dump sur un autre serveur distant et l'applique au serveur distant
### Drop une base de données
