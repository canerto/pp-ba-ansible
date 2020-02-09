# VM einrichten mit Oracle VirtualBox

1. Datei -> Appliance importieren 
2. ITS_Client...ova auswählen
3. Name ändern auf "ITS_Server_X"
4. Häkchen setzen bei "Zuweisen neuer MAC-Adressen für alle Netzwerkkarten"
5. Importieren
6. VM starten, anmelden als "itsadmin" und Konsole starten
7. Folgende Befehle eingeben, um Hostname und IP der VM umzustellen
8. Falls Fehler auftritt => Change Network Settings => OK

itsserver1 10.0.0.11 1 Core, 2GB

itsserver2 10.0.0.21 1 Core, 2 GB

itsserver3 10.0.0.31 1 Core, 2 GB

awxclient  10.0.0.41 2 Core, 4 GB

itsclient  10.0.0.51 1 Core 2 GB

```
-- itsclient
sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 itsclient.itsdomain.local itsclient
10.0.0.11 itsserver1.itsdomain.local itsserver1
10.0.0.21 itsserver2.itsdomain.local itsserver2
10.0.0.31 itsserver3.itsdomain.local itsserver3
10.0.0.41 awxclient.itsdomain.local awxclient
10.0.0.51 itsclient.itsdomain.local itsclient
EOF
sudo /sbin/reboot

--itsserver1
$ export my_hostname=itsserver1
$ export my_ip_suffix=11

$ if [ -z "${my_hostname}" ] ; then echo "Variable my_hostname nicht gesetzt!" ; fi
$ if [ -z "${my_ip_suffix}" ] ; then echo "Variable my_ip_suffix nicht gesetzt!" ; fi
$ sudo /bin/sed -i "s/10.0.0.51/10.0.0.${my_ip_suffix}/g" /etc/network/interfaces 
$ sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 ${my_hostname}.itsdomain.local ${my_hostname} 
10.0.0.11 itsserver1.itsdomain.local itsserver1
10.0.0.21 itsserver2.itsdomain.local itsserver2
10.0.0.31 itsserver3.itsdomain.local itsserver3
10.0.0.41 awxclient.itsdomain.local awxclient
10.0.0.51 itsclient.itsdomain.local itsclient
EOF

$ sudo tee /etc/hostname <<EOF
${my_hostname}
EOF
$ sudo /sbin/reboot

--itsserver2
$ export my_hostname=itsserver2
$ export my_ip_suffix=21

$ if [ -z "${my_hostname}" ] ; then echo "Variable my_hostname nicht gesetzt!" ; fi
$ if [ -z "${my_ip_suffix}" ] ; then echo "Variable my_ip_suffix nicht gesetzt!" ; fi
$ sudo /bin/sed -i "s/10.0.0.51/10.0.0.${my_ip_suffix}/g" /etc/network/interfaces 
$ sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 ${my_hostname}.itsdomain.local ${my_hostname} 
10.0.0.11 itsserver1.itsdomain.local itsserver1
10.0.0.21 itsserver2.itsdomain.local itsserver2
10.0.0.31 itsserver3.itsdomain.local itsserver3
10.0.0.41 awxclient.itsdomain.local awxclient
10.0.0.51 itsclient.itsdomain.local itsclient
EOF

$ sudo tee /etc/hostname <<EOF
${my_hostname}
EOF
$ sudo /sbin/reboot


--itsserver3

$ export my_hostname=itsserver3
$ export my_ip_suffix=31

$ if [ -z "${my_hostname}" ] ; then echo "Variable my_hostname nicht gesetzt!" ; fi
$ if [ -z "${my_ip_suffix}" ] ; then echo "Variable my_ip_suffix nicht gesetzt!" ; fi
$ sudo /bin/sed -i "s/10.0.0.51/10.0.0.${my_ip_suffix}/g" /etc/network/interfaces 
$ sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 ${my_hostname}.itsdomain.local ${my_hostname} 
10.0.0.11 itsserver1.itsdomain.local itsserver1
10.0.0.21 itsserver2.itsdomain.local itsserver2
10.0.0.31 itsserver3.itsdomain.local itsserver3
10.0.0.41 awxclient.itsdomain.local awxclient
10.0.0.51 itsclient.itsdomain.local itsclient
EOF

$ sudo tee /etc/hostname <<EOF
${my_hostname}
EOF
$ sudo /sbin/reboot

--awxclient
$ export my_hostname=awxclient
$ export my_ip_suffix=41

$ if [ -z "${my_hostname}" ] ; then echo "Variable my_hostname nicht gesetzt!" ; fi
$ if [ -z "${my_ip_suffix}" ] ; then echo "Variable my_ip_suffix nicht gesetzt!" ; fi
$ sudo /bin/sed -i "s/10.0.0.51/10.0.0.${my_ip_suffix}/g" /etc/network/interfaces 
$ sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 ${my_hostname}.itsdomain.local ${my_hostname} 
10.0.0.11 itsserver1.itsdomain.local itsserver1
10.0.0.21 itsserver2.itsdomain.local itsserver2
10.0.0.31 itsserver3.itsdomain.local itsserver3
10.0.0.41 awxclient.itsdomain.local awxclient
10.0.0.51 itsclient.itsdomain.local itsclient
EOF

$ sudo tee /etc/hostname <<EOF
${my_hostname}
EOF
$ sudo /sbin/reboot
```




# Ansible

## Grundlagen

### Control Node

Man spricht von Control Nodes, wenn Ansible auf einer Maschine installiert ist. Hierfür kann man jeden Computer nutzen, auf dem Python installiert ist.
Von hier aus kann man Playbooks und Aufgaben laufen lassen. Außerdem kann man mehrere Control Nodes haben, jedoch kann man kein Computer mit dem Windowsbetriebssystem als Control Node nutzen.

### Managed Nodes

Managed Nodes, auch "Hosts" genannt, sind die Geräte, die verwaltet werden.

### Configuration Files

Foto Hierarchie der Konfigs

Die Konfigurationsdatei `/etc/ansible/ansible.cfg` wird nach der Installation von Ansible angelegt. Diese primäre Datei beinhaltet
Default-Einstellungen, z.B. den Pfad zur Inventory.

Eine Beispieldatei hierfür ist verfügbar in GitHub unter https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg und weitere Informationen
sind verfügbar unter dem offiziellen Ansible-Link https://docs.ansible.com/ansible/latest/cli/ansible-config.html#ansible-config


### Inventory

Das Inventory File, meistens auch "Hostfile" genannt, liegt unter `/etc/ansible/hosts`. Der Pfad hierzu kann in der Konfigurationsdatei ansible.cfg geändert werden. 
In dieser Datei wird eine Liste definiert, in der die zu verwaltenden Hosts von Ansible aufgelistet werden.

Beispiel:

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

Die Überschriften in den eckigen Klammern sind Gruppennamen, um mehrere Hosts als eine Gruppe zu definieren.

### Modules

### Ad-hoc Tasks

Ein Ad-hoc Task nutzt die CLI um einzelne Aufgaben auf den Managed Nodes ausführen zu können. Hiermit kann man einfache und schnelle Kommandos ausführen,
haben jedoch den Nachteil, dass Sie nicht wieder nutzbar sind, so wie man es von Playbooks kennt.

### Playbooks

Hierbei handelt es sich um eine Datei, die mehrere Aufgaben/Tasks enthält. Diese werden meist im YAML-Format geschrieben
und sind einfach zu lesen, zu schreiben und zu verstehen.


### Roles
Eine Role ist eine Sammlung von mehreren Playbooks, die notwendig sind um ein Zielszenario zu erreichen, z.B. ein Webserver installieren.

## Einführung

### Installation Control Node unter Debian

Um die aktuellste Version von Ansible für Ubuntu zu nutzen, kann man die PPA (personal package archive) dem System hinzufügen.

Dafür benötigt man zuerst das `software-properties-common` Paket, um Software Repos einfacher zu managen. 
Anschließend das Ansible PPA hinzufügen und dann Ansible installieren.

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
$ ansible --version
```

### Ordnerstruktur Best Practice https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

```
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
   group1.yml             # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```

### Konfiguration Ansible

Die allgemeinen Einstellungen sind in der `ansible.cfg` Datei unter dem Pfad `/etc/ansible/` zu finden. Wir benötigen nun die Inventory-Datei `hosts` im gleichen Ordnern.
Der Pfad hierfür ist per Default eingestellt auf `/etc/ansible/hosts`, welcher in der `ansible.cfg` Datei geändert werden kann. 

Kopieren der Dateien von /etc/ansible in den eigenen erstellten ansible-Ordner, da per default dort installiert wird

```
$ sudo cp -r /etc/ansible .
$ sudo chown -R itsadmin:itsadmin
```

In ansible.cfg den Inventory-Pfad ändern 

```
$ vim ansible.cfg

inventory = ./hosts
```

Wir öffnen nun die Datei wie folgt: (sudo, weil etc root gehört)

```
$ sudo vim ansible/hosts
```

Hier sind bereits einige Beispiele enthalten. Man kann die Hosts nun als IP-Adressen, DNS-Namen oder als eigene Namen definieren.
Wir fügen nun folgende Zeilen hinzu und speichern anschließend die Datei:

```
$ itsserver1 ansible_host=10.0.0.11 
$ itsserver2 ansible_host=10.0.0.21 
$ itsserver3 ansible_host=10.0.0.31 
```

Wir haben nun die Hosts angegeben mit der IP-Adresse. 

Nun können wir testen, ob alles richtig konfiguriert wurde, in dem wir zuerst einmal vesuchen den ersten Host zu 
pingen und schauen, welche Rückmeldung wir erhalten:

```
$ ansible -m ping itsserver1
```

Erste Verbindung, deshalb die Frage:
'The authenticity of host '10.0.0.11 (10.0.0.11)' can't be established. ECDSA key fingerprint is ... '
Are you sure you want to continue connecting (yes/no)?

 
ERWARTUNG:

```
itsserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

FEHLERMELDUNG:

```
itsserver1 | UNREACHABLE => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Warning: Permanently added '10.0.0.11 (ECDSA) to the list of known hosts. 
     Permission denid (publickey, password). ",
    "unreachable": true
}
```

Hinweis: 
```
A trivial test module, this module always returns pong on successful contact. It does not make sense in playbooks, but it is useful from /usr/bin/ansible to verify the ability to login and that a usable python is configured.
This is NOT ICMP ping, this is just a trivial test module.
For Windows targets, use the win_ping module instead.
```

Dieser Befehl bezieht sich auf einen in der Inventory-Datei befindenden Hosts. Wenn wir nun alle Hosts anpingen möchten,
können wir folgenden Befehl nutzen:

```
$ ansible -m ping all
``` 

Als Ausgabe sollten wir folgendes erhalten (zuerst SSH)

```
itsserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

itsserver2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

itsserver3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

itsserver4 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## SSH-Konfiguration

Ansible kommuniziert primär mit den Managed Nodes via SSH. Zuerst erzeugen wir ein SSH-Schlüsselpaar auf dem Control Node,
um später eine passwortlose Anmeldung auf den Managed Nodes zu ermöglichen.

```
$ ssh-keygen
```

Weitere Optionen....

```
$ ssh-keygen -t rsa -b 2048
```

Die folgenden Fragen können wir mit Return bestätigen. Dadurch entstehen nun im /.ssh/ Verzeichnis zwei Dateien. Es handelt sich um 
den private und public Key.


## SSH-Key-Verteilung 

### explizit für einen Node
Der public Key muss nun auf die zu verwaltenden Nodes kopiert werden.

```
$ cd .ssh/
$ ssh-copy-id itsadmin@<entfernter Node>
```

Im Anschluss versuchen wir eine SSH-Verbindung auf den zu verwaltenden Node aufzubauen, ohne ein Passwort eingeben zu müssen.

```
$ ssh root@<entfernter Node>
```

PING => FEHLERMELDUNG PYTHON3, dazu kommen wir aber später

### mit einem Playbook für alle Nodes

1. Wir erstellen dazu einen Ordner roles, in dem alle Rollen enthalten sein werden und die dazugehörigen Playbooks

```
$ mkdir roles
$ cd roles
$ mkdir deploy_ssh_keys
$ cd deploy_ssh_keys
$ mkdir tasks
$ cd tasks
$ vim main.yml 
```

In main.yml folgendes einfügen:

```
 - name: Ensure key is in root's ~/.ssh/authorized_keys     # Beschreibung
   authorized_key:                                          # Modul für authorized_key
    user: root                                              # Remoteuser
    state: present                                          # present, da die Datei hinzugefügt werden soll (sonst absent)
    key: '{{ item }}'                                       
   with_file:
    - ~/.ssh/id_rsa.pub
```

Im Anschluss im Hauptverzeichnis (gleiche Ebene wie roles) eine `main.yml` erstellen, da wir in dieser Hauptdatei, alle Rollen aufrufen werden.

```
$ vim main.yml
```

In main.yml folgendes einfügen:

```
- - -
- name: Update all packages, deploy SSH Keys to Managed Nodes 
  hosts: all
  become: yes
  become_user: root
  become_method: sudo
  tasks:
    - name: Update all packages
      apt:
       upgrade: dist
   
  roles:
    - deploy_ssh_keys
    
```
TODO: Erklärung der einzelnen Zeilen

Hiermit deployen wir die SSH-Keys für Nodes 2-4. Dies testen wir nun, indem wir folgendes ausführen:
 
```
ansible-playbook <playbook-name>
```

Wir bekommen die Fehlermeldung, dass wir keinen Zugriff haben (Permission denied), da hierfür Root-Rechte benötigt werden
und somit beim Erstkontakt keine Authentifizierung ohne Passwort stattfinden kann. Dazu muss einmalig folgender Befehl ausgeführt
werden und das Passwort angegeben werden, wenn dies angefordert wird.


```
ansible-playbook <playbook-name> --ask-pass
```

Alternativ:
Alternativ kann man in der Inventory angeben, welchen User wir verwenden und welches Passwort. Das Passwort sollte normalerweise 
nicht als Klartext eingefügt werden. Hierfür gibt es Ansible Vault, womit Passwörter verschlüsselt werden können. 
Dazu kann man die Variablen in group_vars referenzieren und anschließend mit Ansible Vault verschlüsseln. Ein Beispiel hierfür
wird im nächsten Kapitel thematisiert.

```
$ itsserver1 ansible_host=10.0.0.11 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
$ itsserver2 ansible_host=10.0.0.12 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin 
$ itsserver3 ansible_host=10.0.0.13 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
$ itsserver4 ansible_host=10.0.0.14 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
```

TODO: Unterschied von manueller Verteilung für ein Host und Verteilung mit einem Playbook für mehrere Hosts

# Python installieren/Prüfung, ob neueste Version installiert ist (notwendig auf allen Managed Nodes)
=> [DEPRECATION WARNING]: Distribution Ubuntu 16.04 on host itsserver should use /usr/bin/python3, but is using /usr/bin/python
for backward compatibility with prior Ansible releases. A future Ansible release will defaulto to using the discoverd platform
python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
This feature will be removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in
ansible.cfg

1.Neue Rolle erstellen 

```
$ cd roles
$ mkdir pyhton
$ mkdir tasks
$ vim main.yml
```

2.Code einfügen

```
- name: Install python
  apt:
    name: python3-pip
    state: latest
    update_cache: yes
```
    
3.Als Rolle einfügen in der Hauptdatei und anschließend ausführen

```
- - -
- name: Update all packages, deploy SSH Keys to Managed Nodes 
  hosts: all
  become: yes
  become_user: root
  become_method: sudo
  tasks:
    - name: Update all packages
      apt:
       upgrade: dist
   
  roles:
    - deploy_ssh_keys
    - python
    
```

# User erstellen mit Variablen und Passwörter in vars datei mit Ansible Vault verschlüsseln

Wir erstellen nun eine neue Rolle "add user" um einen User zu erstellen.

```
$ cd roles
$ mkdir add_user
$ mkdir tasks
$ vim main.yml
```

Anschließend fügen wir folgenden Code hinzu:

```
- name: Add user
  hosts: all
  tasks:
    - name: Add a new user with username and commentname
      user:
        name: "{{ username }}"
        comment: "{{ name }}"
```

Hier wurden nun die Variablen "{{ username }}" und "{{ name }}" eingefügt. Das bedeutet, dass man diesem Task eigene Namen übergenen kann.
Dies kann wie folgt durchgeführt werden:

```
ansible-playbook main.yml -e "username=test name=test"
```

Nun erstellen wir einen Ordner /vars und fügen hier die Datei main.yml hinzu, in der wir die Liste unserer User mit Passwort definieren.
Anschließend können wir diese Liste in unserem Task aufrufen.

In /vars/main.yml

```
# A list of users who will be added
users:
  - username: user1
    name: "User1"
    password: "user1"
  - username: user2
    name: "User2"
    password: "user2"
```

In /tasks/main.yml
 
```
- include_vars: ../vars/main.yml
- name: Add a new user with username and commentname who are defined in vars, if the account doesn't exist
  user:
    name: "{{ item.username }}"
    state: present
    comment: "{{ item.name }}"
    password: "{{ item.password | password_hash('sha512') }}"
   with_items: "{{ users }}"
   when: users | length > 0
```

Das Passwort muss gehasht werden, da sonst das Passwort bzw. der Login nicht funktioniert. Die Variablen werden aus der
vars/main.yml gezogen. Die Passwörter in dieser Datei sind jedoch sichtbar und in Plaintext geschrieben.
Dies wollen wir verhindern, in dem wir Ansible Vault nutzen. Dazu erstellen wir im Ordner vars eine verschlüsselte Datei
 
 `ansible-vault create vault`
 
Hier definieren wir unsere Passwörter unserer User:

```
- - -
vault_user1_password: user1
vault_user2_password: user2
```

In vars/main.yml:

```
# A list of users who will be added
users:
  - username: user1
    name: "User1"
    password: "{{ vault_user1_password }}"
  - username: user2
    name: "User2"
    password: "{{ vault_user2_password }}"
```

# Software Docker installieren und Container automatisiert erstellen

1.Wir erstellen nun eine neue Rolle "docker", diesmal mit zwei Dateien, da wir eine Installations-Playbook 
`install_docker.yml` erstellen und diese dann in die `main.yml` imkludieren.

```
$ cd roles
$ mkdir docker
$ mkdir tasks
$ touch install_docker.yml
$ touch main.yml
```

2. Installation von Docker, dazu folgenden Code in `install_docker.yml` einfügen:
```
- - -
  - name: Ensure old versions of Docker are not installed
    apt:
      name:
        - docker
        - docker-common
        - docker-engine
      state: absent

  - name: Ensure dependencies are installed
    apt:
      name: 
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg-agent
        - software-properties-common
      state: present

  - name: Add Docker's official GPG key
    apt_key: 
      url: https://download.docker.com/linux/ubuntu/gpg
      id: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88
      state: present

  - name: Verify the fingerprint
    shell: sudo apt-key fingerprint 0EBFCD88
    register: fingerprint

  - debug: var=fingerprint

  - name: Add Docker repository
    apt_repository:
      repo: "{{ docker_apt_repository }}"
      state: present

  - name: Install Docker
    apt:
      name:
        - docker-ce
        - docker-ce-cli
        - containerd.io
      state: present
      update_cache: yes
    notify: restart docker

  - name: Ensure Docker is started and enabled at boot
    service:
      name: docker
      state: started
      enabled: yes

  - name: Verify that Docker Engine is installed correctly by running hello-world image
    shell: sudo docker run hello-world
    register: hello

  - debug: var=hello

  - name: Manage Docker as a non-root user, so ensure docker users are added to the docker group
    user:                                       
      name: "{{ item }}"
      group: docker 
      append: yes
    with_items: "{{ docker_users }}"
    when: docker_users | length > 0

  - name: Install Docker Module for Python
    pip:
      name: docker

  - name: Reboot the machine
    reboot:
```

TODO: Code erklären

3. Vars erstellen

4. Handlers

5. Erstellen von bel. Anzahl von Containern mit Ubuntu-Images, dazu folgenden Code in `main.yml` einfügen:
```
- - -
  - import_tasks: install_docker.yml
  - include_vars: ../vars/main.yml
  - name: Pull image test with ubuntu:latest 
    docker_image:
      name: ubuntu
      source: pull

  - name: Create ubuntu container with variable numbers
    docker_container:
      name: "ubuntu{{item}}"
      image: ubuntu:latest
      state: started
      command: sleep 1d
    with_sequence: count=4

  - name: List running containers
    shell: docker ps -a
    register: container

  - debug: var=container
```
TODO: Code erklären

# Rolle für Webserver erstellen mit LAMP TODO

1.Wir erstellen nun eine neue Rolle "lamp"

```
$ cd roles
$ mkdir lamp
$ mkdir tasks
$ vim main.yml
```

2. Folgenden Code einfügen:

```
# Install MySQL, Apache, PHP
  - name: Check package if installed and installs if not present
    apt:
      pkg:
        - "mysql-server"
        - "apache2"
        - "php"
        - "php-mysql"
        - "libapache2-mod-php"
      update_cache: yes
      cache_valid_time: 3600
      state: present

  - name: Check if mysql is running and enable to start on boot
    service:
      name: mysql
      state: started
      enabled: yes

  - name: Check if apache2 is running and enable to start on boot
    service:
      name: apache2
      state: started
      enabled: yes
```

3.Endversion Playbook mit allen Roles
```
- - -
- name: Update all packages, deploy SSH Keys to Managed Nodes, Install Python, Add a new User, Install Docker and Create Container
  hosts: all
  become: yes
  become_user: root
  become_method: sudo
  tasks:
    - name: Update all packages
      apt:
       upgrade: dist
   
  roles:
    - deploy_ssh_keys
    - python
    - add_user
    - docker

- name: Install LAMP (MySQL, Apache, PHP) on webservers
  hosts: webservers
  become: yes
  become_user: root
  become_method: sudo

  roles:
    - lamp
```

4.Test apache => localhost im Browser aufrufen

5.Test php => phpinfo.php in /var/www/html/ erstellen mit folgendem Inhalt:

```
<?php
phpinfo();
?>
```

6.localhost/phpinfo.php aufrufen


# Ansible Vault

1.Möglichkeit:
Anlegen einer Datei, Passworteingabe und Text:

```
ansible-vault create vault.yml
```

Testen mit cat vault.yml:

2.Möglichkeit: 
Bestehende Datei verschlüsseln und Passworteingabe
```
ansible-vault encrypt existing_file.yml
```

=> Inhalt sehen mit ansible-vault view vault.yml
=> Bearbeiten mit ansible-vault edit vault.yml
=> PW ändern mit ansible-vault rekey vault.yml
=> ansible-playbook --ask-vault-pass
=> Passworddatei erstellen und diese referenzieren, ansible-playbook --vault-password-file=path
=> Passworddatei automatisch lesen in dem environment variable gesetzt wird und in der Konfig angeben
```
export ANSIBLE_VAULT_PASSWORD_FILE=PATH
vim ansible.cfg
```

In `ansible.cfg`:
```
...
vault_password_file = path
```
# Ansible AWX

# Ansible Galaxy

# Ansible Doc