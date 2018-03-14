# Ansible Command line et Playbooks
## Introduction
Introduction à Ansible via des exemples d'utilisation en mode ligne de commande et en mode playbook.

Pour exécuter les commandes et les playbook, il vous faudra trois machines virtuelles, une servant de maitre (celle sur laquelle Ansible est installé) et deux machines virtuelles esclaves servant de serveur remote auxquels ansible va se connecter via SSH.

Dans nos exemples les trois serveurs utilisée sont:
 * Debian Ansible Server: 192.168.56.100
 * Debian 1 (un serveur remote): 192.168.56.101
 * Debian 2 (un serveur remote): 192.168.56.102

## Installation
### VM VirtualBox
1. Créer un "Host-Only Ethernet Adapter" sur lequel les trois serveurs vont se connecter.
2. Dans les propriétés de chaque machine virtuelle, aller dans l'onglet "Network" et ajouter un "Adapter" de type "Host-only Adapter". Selectionner à l'aide de la liste déroulante l'adapteur créé lors de l'étape 1 pour le champ "Name".

### Pour tous les serveurs
1. Créer une clé ssh: `ssh-keygen`
2. Ajouter une interface réseau a votre serveur avec une ip différente pour chacun d'entre eux. Pour ce fait, éditer `/etc/network/interfaces` et ajouter l'interface enp0s8: 
``` 
auto enp0s8
iface enp0s8 inet static
address 192.168.56.100
netmask 255.255.255.0
```
Ici nous assignons au serveur l'ip `192.168.56.100`.  
3. Editer le fichier `/etc/hostname` pour donner un nom à votre machine. Dans nos exemples nous avons: 
* ansible-server (ip: 192.168.56.100)
* debian-1 (ip: 192.168.56.101)
* debians-2 (ip: 192.168.56.102)
4. Editer le fichier `etc/hosts` en y ajoutant l'ip du serveur suivi de son hostname. Exemple: `192.168.56.100 ansible-server`.

### Pour le serveur Ansible
Dans cet exemple le serveur Ansible est un serveur Debian ayant l'ip: 192.168.56.100.
1. Installer Ansible: [Instructions ici](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-debian)
2. Ajouter les serveurs remote au fichier `/etc/ansible/hosts`.
Nous créons trois groupes de serveurs. Nos deux serveurs remote (192.168.56.101 et 192.168.56.102) seront des serveurs de BD. Afin d'exécuter des commandes sur tous les serveurs de BD à la fois, nous créons un groupe **dbservers** contenant les ip de nos deux serveurs. Deux groupes **debians1** et **debian2** permettenent d'acceder à chaque serveur remote individuellement.
 
```
[dbservers]
192.168.56.101
192.168.56.102

[debian1]
192.168.56.101

[debian2]
192.168.56.102
```
3. Partager la cle publique SSH du serveur Ansible avec les remote:  
`ssh-copy-id -i ~/.ssh/id_rsa <votre_user_debian1>@192.168.556.101`  
`ssh-copy-id -i ~/.ssh/id_rsa <votre_user_debian2>@192.168.556.102`

Votre serveur Ansible est désormait pret à etre utilisé!!

## Ansible: Command Line
Il est possible d'utiliser Ansible pour effectuer de simples manipulations sur des serveurs remote via un terminal et des lignes de commandes. La syntaxe de base est la suivante:  
`ansible <host_groupe> -m <nom_du_module> -a <arguments_du_module>`.  
Voici une liste non exhaustives des opérations possibles de faire:

### Exécuter une commande Shell
Exécuter la commande shell `cat /etc/hostname` sur tous les serveurs BD (`dbservers`)  afin de retourner le hostname de chaque serveur:  
`ansible dbservers -m shell -a 'cat /etc/hostname'`  
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
Cette commande va changer la permission sur les serveurs remote du fichier au path spécifié:  
`ansible dbservers -m file -a "dest=/tmp/test.txt mode=777"`  
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
Note: On peut aussi changer l'owner et le groupe du fichier via cette commande

### Créer un dossier
Cette commande créée la structure de dossiers `dir_from/ansible/` sur les serveurs remote à l'intérieur du dossier `/tmp` des serveurs remote. De plus, ces dossiers auront le mode, l'owner et le groupe dont les valeurs sont spécifiées en argument du module.  
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
Cette commande supprime le dossier spécifié en argument ainsi que son contenu.
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

### S'assurer qu'un paquet est installé
Cette commande vérifie que le paquet sudo est bien installé sur le serveur remote
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
Note: On peut aussi installer ou désinstaller des paquets via ce module en utilisant l'argument `state=present` ou `state=absent`

## Ansible: Playbook

Ansible Playbook sont des fichiers YAML permettant d'exécuter une succession de taches sur un ou plusieurs groupes de serveurs.
Exemple de playbook avec une tache consernant les hosts `dbservers`.
```yaml
---
- hosts: dbservers		# <== nom du groupe de serveurs sur lesquels opérer
  tasks:		# <== liste des taches à exécuter sur les serveurs du groupe
    - name: Create a new file in /tmp 	# <== Nom de la tache
      file:								# <== Nom du module utilisé
        path: /tmp/test_no_var.txt 		# <== Argument path du module file et sa valeur 
        state: touch					# <== Argument state du module file et sa valeur 
```
### Exécuter un playbook
Pour exécuter un playbook, faites:  
`ansible-playbook my_play_book.yaml [-K]`  
L'option **-K** est à ajouter losque l'on veut se connecter en temps que root sur un serveur remote afin de transmettre le mot de passe root.

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
Ce playbook introduit la notion de variables. Il est possible de définir des variables dans un playbook pour ensuite les utiliser dans une tâche ou dans un template. Ce playbook va créer le fichier `file_from_ansible_variable_name.txt` dans le dossier `/tmp` du serveur remote.
```yaml
---
- hosts: dbservers
  vars :  # <== On liste ci dessous les variables
    file_name: file_from_ansible_variable_name.txt 
  tasks:
    - name: Create a new file in /tmp
      file:
        path: /tmp/{{ file_name }}
        state: touch
```
### Créer un fichier depuis un template
Ce playbook initialise des variables et créé un fichier `index.html` dans le dossier `/tmp` des serveurs remote. Le fichier `index.html` est basé sur le template situé dans le dossier `templates/`. Le template est une page HTML dans laquelle on va remplacer `{{ page_title }}` et `{{ page_body }}` par les valeurs des variables initialisées dans la partie **vars** du playbook.
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

### Créer un fichier sur un serveur remote et le copier sur un autre serveur distant
Ce playbook se connecte sur le groupe debian1(composé d'une seule machine) pour créer un fichier. Ensuite, Ansible se connecte au groupe debian2 (composé lui aussi d'une seule machine) et va se faire envoyer par la machine du groupe debian1 le fichier `file_from_debian1.txt`. Ce fichier en provenance de debian1 (car la tache est déléguée à debian1 `delegate_to: "{{ groups.debian1[0] }}"`) va être copier sur la machine courante (debian2) dans le dossier `/tmp`.
```yaml
---
- hosts: debian1
  vars :
    filename: file_from_debian1.txt
  tasks:
    - name: Create a file to be copy to debian2 later on 
      file:
        path: /tmp/{{ filename }}
        state: touch

- hosts: debian2
  vars :
    filename: file_from_debian1.txt
  tasks:
    - name: Move file from debian1 to debian 2
      synchronize:
        src: /tmp/{{ filename }}
        dest: /tmp
      delegate_to: "{{ groups.debian1[0] }}"
```

### Un serveur distant ping un autre serveur distant
Ce playbook se connecte sur la machine du groupe debian2 et délègue la tâche de ping à la machine du groupe debian1. La machine du groupe debian1 ping donc la machine du groupe debian2.
```yaml
---
- hosts: debian2
  tasks:
    - name: Ping debian2 from debian1
      ping: 
      delegate_to: "{{ groups.debian1[0] }}" 
```

## Ansible roles
Les roles ansible permettent d'effectuer des configurations de serveur facilement réutilisable. Cela permet entre autre d'installer des packages, créer des fichiers de configuration, changer des permissions sur des fichiers et/ou dossiers, etc.  
Visiter [cette page](http://docs.ansible.com/ansible/latest/playbooks_reuse_roles.html) pour plus d'informations.
### Common role
**Common** role est role que je veux appliquer à tous mes serveurs. Ce role installe des packages de base comme rsync pour que les serveurs puissent transferer des fichiers entre eux.  
Definition des variables dans `roles/common/defaults/main.yaml`. La liste de paquets à installer est défini dans ce fichier.
```yaml
packages:
  - rsync
  - iputils-ping
```
Définition des taches à exécuter pour ce role dans `roles/common/tasks/main.yaml`. La seule tache consiste à installer tous les paquets via apt.
```yaml
- name: install packages defined in defaults/main.yaml
  apt:
    name: "{{ item }}"
    state: present
  with_items: "{{ packages }}"
```
### MySQL role
Le role MySQL sert a installer et configurer MySQL sur le serveur.
* Definition des variables dans `roles/mysql/defaults/main.yaml`. La variable sert a définir le mot de passe root.
```yaml
mysql_root_password: root
```
* Stockage des templates à utiliser pour ce role dans `roles/mysql/templates`. Ce role dispose du template `.my.cnf.j2` permettant de générer un fichier `.my.cnf` avec la variable **mysql_root_password** du role.
```cnf
[client]
port		= 3306
socket		= /var/run/mysqld/mysqld.sock
user 		= root
password 	= {{mysql_root_password}}
```
* Stockage des fichiers utilisés poru le role dans `roles/mysql/files`. Le role **mysql** a une tache de création d'une base de données et d'impostation de données dans celle-ci. Cette tache se fait via l'excution de scripts SQL stockés dans `roles/mysql/files/test_db`.  

* Définition des taches à exécuter pour ce role dans `roles/mysql/tasks/main.yaml`. De nombreuses taches sont effectuées pour installer et configurer le serveur. La description de chaqune d'entre elle est donné par son attribut `name`:
```yaml
- name: Install MySQL required packages
  apt:
    name: "{{item}}" 
    state: present
  with_items:
    - mysql-server
    - mysql-client
    - python-mysqldb

- name: Start the MySQL service
  action: service name=mysql state=started 

- name: Remove the test database
  mysql_db: name=test state=absent

- name: Create deploy user for mysql
  mysql_user: user="deploy" host="%" password={{mysql_root_password}} priv=*.*:ALL,GRANT

- name: Ensure anonymous users are not in the database
  mysql_user: user='' host={{item}} state=absent
  with_items:
    - 127.0.0.1
    - ::1
    - localhost

- name: Copy .my.cnf file with root password credentials
  template: src=.my.cnf.j2 dest=/etc/mysql/my.cnf owner=root mode=0600

- name: Update mysql root password for all root accounts
  mysql_user: name=root host={{item}} password={{mysql_root_password}}
  with_items:
    - 127.0.0.1
    - ::1
    - localhost

- name: Copy the folder that contains sql files from control to remote
  copy:
    src: test_db
    dest: /tmp

- name: Import employees.sql similar to mysql -u <username> -p <password> < hostname.sql
  mysql_db:
    state: import
    name: all
    target: /tmp/test_db/employees.sql
```
### Exemples de playbook utilisant un role
#### Copier un fichier d'un serveur distant à un autre serveur distant
Ce playbook applique le **common** role aux serveurs remote afin qu'ils aient les paquets nécessaires pour faire la synchronisation de fichier (package rsync) et effectue ensuite la copie d'un fichier du serveur debian1 au serveur debian2.
```yaml
---
- hosts: dbservers
  become: true
  roles:
    - common

- hosts: debian2
  vars :
    filename: ansible_deb.txt
  tasks:
    - name: Move file from debian1 to debian 2
      synchronize:
        src: /tmp/{{ filename }}
        dest: /tmp
      delegate_to: "{{ groups.debian1[0] }}"
``` 

#### Dump une base de données
Ce playbook permet de faire un dump d'une base de donnée MySQL. Dans notre cas on fait un dump de la base de données **employees** et le sauvegardons dans `/tmp/dump_employees.sql`. On peut noter la présence du role **mysql** dans ce playbook, cela veut dire que le serveur se vera installer et configurer MySQL si ce n'est pas déja fait. On utilise l'option `become: true` afin qu'ansible se connecte en temps que root sur le serveur remote. Il faudra donc exécuter ce playbook en utilisant l'option **-K** et entrer le mot de passe root lorsque demandé.
```yaml
---
- hosts: dbservers
  roles:
    - mysql
  become: true
  vars:
    dump_file_name: dump_employees.sql
  tasks:
    - name: Creates a dump of the employees db and store it in /tmp
      mysql_db:
        state: dump
        name: employees
        target: /tmp/{{dump_file_name}}
```
#### Applique un dump sur un serveur distant
Ce playbook. S'assure que le serveur a bien un role mysql, se connecte sur debian1 pour ajouter des données dans la BD afin dàvoir une version différente de cette de la BD de debian2. Un dump de tous les serveurs de base de données (debian1 et debian2) est fait. Le dump de debians1 est ensuite copié dans debian2 qui va l'appliquer à sa base de données pour avoir une version identique à celle de debian1.
```yaml
---
- hosts: dbservers
  become: true
  roles:
    - mysql

- hosts: debian1
  become: true
  tasks:
    - name: Add more data to employees db to have a different data than Debian2's employees db
      mysql_db:  
        state: import
        name: employees
        target: /tmp/test_db/extra_employees.dump

- hosts: dbservers
  become: true
  vars:
    dump_file_name: dump_employees
    info_host: "{{ ansible_fqdn }}"
  tasks:
    - name: Creates a dump of the employees db and store it in /tmp
      mysql_db:
        state: dump
        name: employees
        target: /tmp/{{ dump_file_name }}_{{ info_host }}.sql

- hosts: debian1
  vars:
    dump_file_name: dump_employees
    info_host: "{{ ansible_fqdn }}"
  tasks:
    - name: Copy dump deb1 to local (deb2)
      synchronize:
        src: /tmp/{{ dump_file_name }}_{{ info_host }}.sql
        dest: /tmp
        mode: pull
      delegate_to: "{{ groups.debian2[0] }}"
    - name: Apply dump
      become: true
      mysql_db:
        state: import
        name: employees
        target: /tmp/{{ dump_file_name }}_{{ info_host }}.sql
      delegate_to: "{{ groups.debian2[0] }}"
```
### Drop une base de données
Ce playbook permet d'effacer une base de données. Dans notre cas, nous supprimons la base de données **employee**.
```yaml
---
- hosts: dbservers
  roles:
    - mysql
  become: true
  vars:
    dump_file_name: dump_employees
    info_host: "{{ ansible_fqdn }}"
  tasks:
    - name: Delete employees database
      mysql_db:
        name: employees
        state: absent
    - name: Delete dump in /tmp
      file: 
        path: /tmp/{{ dump_file_name }}_{{ info_host }}.sql
        state: absent
    - name: Delete test_db folder
      file: 
        path: /tmp/test_db
        state: absent
```