
# Biblio

Note de la formation : https://pad.opendoor.fr/p/SIB20231102

TMUX : https://infra.opendoor.fr/git/tom/tmux_formation

VIM : https://infra.opendoor.fr/git/tom/vim_formation

STARSHIP: https://starship.rs/guide/#%F0%9F%9A%80-installation

Depot git du formateur : https://infra.opendoor.fr/git/tom {sib_*_}
* https://cours.opendoor.fr/Fichiers/SIB
* https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin
* https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html


Ansible.cfg

Attention à l'emplacement du fichier ansible.cfg

>Le premier qui est trouvé, dans l'ordre de recherche suivant, est utilisé, tous les
autres sont ignorés :
1/ Emplacement spécifié par la variable ANSIBLE_CONFIG,
2/ ${PWD}/ansible.cfg,
3/ ${HOME}/.ansible.cfg,
4/ /etc/ansible/ansible.cfg.
Son format est similaire aux fic

```ini

[defaults]
forks                    = 20 # nombre de connexion simultané
ask_pass                 = True
host_key_checking        = False
callbacks_enabled        = ansible.posix.timer, ansible.posix.profile_tasks # activation des stat d'execution
remote_user              = formation
gathering                = smart # recupearation intelligente des données des hotes
fact_caching             = jsonfile
fact_caching_connection  = /etc/ansible/facts
fact_caching_timeout     = 86400
retry_files_enabled      = False
strategy                 = free
ansible_managed          = Ansible managed : {file} modified on %Y-%m-%d by {uid} on {host} # entete des fichiers manipuler par ansible
[inventory]
[privilege_escalation]
become                  = True
become_ask_pass         = True
[paramiko_connection]
[ssh_connection]
pipelining              = True
[persistent_connection]
[accelerate]
[selinux]
[colors]
[diff]


```


Lister des plugins
```bash
ansible-doc -l -t inventory
```

Afficher le contenu d'un inventaire
```
ansible-inventory --graph # affichage d'un graph de l'inventaire
ansible-inventory --host # affichage de l'hote abvec la liste des variables associés
```

Commande ADHOC : 

Configuration des VIM Sur toutes les machines en utilisant le module builtin lineinfile
```bash
ansible localhost -m ansible.builtin.lineinfile -a 'path="/etc/vimrc" line="set softtabstop=2 expandtab shiftwidth=2 smarttab autoindent" create=true'
```

Telecharement d'un fichier
```bash
ansible -o all -m ansible.builtin.get_url -a 'url="https://starship.rs/install.sh" dest="/tmp" mode=0755'

```
Execution d'une commande 
```bash
ansible -o all -m ansible.builtin.command -a '/tmp/install.sh --yes creates="/usr/local/bin/starship"' 
```


S'assurer de la présence du fichier /etc/profile.d/zstarship.sh avec le contenu suivant:
```
eval "$(/usr/local/bin/starship init bash)"
```

```
ansible  -o  all -m copy -a 'content="eval \"$(/usr/local/bin/starship init bash)\"" dest=/etc/profile.d/zstarship.sh'
```


## tp sib 6

ansible localhost -m community.crypto.openssh_keypair -a 'path=/home/formation/.ssh/id_ssh_ansible type=ed25519 owner=formation group=formation'

ansible -o all -m ansible.builtin.user -a "name=ansible password={{ '123Soleil'|password_hash('sha512') }} home=/home/ansible create_home=true"

ansible -o all -m ansible.builtin.copy -a 'src="/home/formation/.ssh/id_ssh_ansible" dest="/home/ansible/.ssh/id_ssh_ansible"'


ansible cibles -o -m community.general.sudoers -a 'name="ansible" user=ansible commands=ALL nopassword=true'


# playbook stucture minimum

```yaml

- hosts: ...
  tasks:
  - name: ...
```

# Vérification d'un le syntaxe d'un playbook
```bash
ansible-playbook  fichier.yml --syntax-check
```

# --check lancement d'une simulation
```bash
ansible-playbook  fichier.yml --checksyntax
```

# Etapes de réalisationd d'un playbook
page 52 du PDF de support

1. Décomposer chaque étape de la procédure en une tâche simple,

1. Identifier le module permettant de réaliser l'opération,, par ex pour l'installation
et la configuration de vim :
1. installer le paquet vim-enhanced ⇒ module yum
2. copier le fichier de configuration vers la cible ⇒ module copy
3.Identifier les paramètres des modules à utiliser (documentation),
4/ Rassembler les tâches dans un playbook, et y spécifier :
Le nom
Les machines cibles
Les informations de connexion et de compte.
5/ Lancer l'exécution du playbook avec l’option --syntax-check pour s'assurer de
l'absence d'erreur de syntaxe.
6/ Lancer le playbook en mode simulation (option –check)
7/ Lancer le playbook sur une environnement de test, puis en production.


Les handlers permettent de gere les evenements d'une taches
il s'executre à la reception d'un signal.
voir page 62 du pdf


# JOUR 2
## les roles 

création d'un role 
```
ansible-galaxy init <pseudo.rolename>
```
execution d'un pmlaybook uniquement sur une machine  utilisation du --<limit nom de machine>
