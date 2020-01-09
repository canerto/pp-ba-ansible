# VM einrichten mit Oracle VirtualBox

1. Datei -> Appliance importieren 
2. ITS_Client...ova auswählen
3. Name ändern auf "ITS_Server_X"
4. Häkchen setzen bei "Zuweisen neuer MAC-Adressen für alle Netzwerkkarten"
5. Importieren
6. VM starten, anmelden als "itsadmin" und Konsole starten
7. Folgende Befehle eingeben, um Hostname und IP der VM umzustellen


```
$ export my_hostname=itsserver
$ export my_ip_suffix=11

$ if [ -z "${my_hostname}" ] ; then echo "Variable my_hostname nicht gesetzt!" ; fi
$ if [ -z "${my_ip_suffix}" ] ; then echo "Variable my_ip_suffix nicht gesetzt!" ; fi
$ sudo /bin/sed -i "s/10.0.0.51/10.0.0.${my_ip_suffix}/g" /etc/network/interfaces 
$ sudo tee /etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 ${my_hostname}.itsdomain.local ${my_hostname} 
10.0.0.11 itsserver.itsdomain.local itsserver
10.0.0.51 itsclient.itsdomain.local itsclient
EOF

$ sudo tee /etc/hostname <<EOF
{my_hostname}
EOF
t
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
```

### Ordnerstruktur

--

### Konfiguration Ansible

Die allgemeinen Einstellungen sind in der `ansible.cfg` Datei unter dem Pfad `/etc/ansible/` zu finden. Wir benötigen nun die Inventory-Datei `hosts` im gleichen Ordnern.
Der Pfad hierfür ist per Default eingestellt auf `/etc/ansible/hosts`, welcher in der `ansible.cfg` Datei geändert werden kann. 

Wir öffnen nun die Datei wie folgt: (sudo, weil etc root gehört)

```
$ sudo vim /etc/ansible/hosts
```

Hier sind bereits einige Beispiele enthalten. Man kann die Hosts nun als IP-Adressen, DNS-Namen oder als eigene Namen definieren.
Wir fügen nun folgende Zeilen hinzu und speichern anschließend die Datei:

```
$ itsserver1 ansible_host=10.0.0.11 
$ itsserver2 ansible_host=10.0.0.12 
$ itsserver3 ansible_host=10.0.0.13 
$ itsserver4 ansible_host=10.0.0.14 
```

Wir haben nun die Hosts angegeben mit der IP-Adresse. 

```
A trivial test module, this module always returns pong on successful contact. It does not make sense in playbooks, but it is useful from /usr/bin/ansible to verify the ability to login and that a usable python is configured.
    This is NOT ICMP ping, this is just a trivial test module.
    For Windows targets, use the win_ping module instead.
```

Nun können wir testen, ob alles richtig konfiguriert wurde, in dem wir zuerst einmal vesuchen den ersten Host zu 
pingen und schauen, welche Rückmeldung wir erhalten:

```
$ ansible -m ping itsserver1
```

```
itsserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Dieser Befehl bezieht sich auf einen in der Inventory-Datei befindenden Hosts. Wenn wir nun alle Hosts anpingen möchten,
können wir folgenden Befehl nutzen:

```
$ ansible -m ping all
``` 

Als Ausgabe sollten wir folgendes erhalten:

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
$ cd /root/.ssh/
$ ssh-copyid root@<entfernter Node>
```

Im Anschluss versuchen wir eine SSH-Verbindung auf den zu verwaltenden Node aufzubauen, ohne ein Passwort eingeben zu müssen.

```
$ ssh root@<entfernter Node>
```

### mit einem Playbook für alle Nodes

1. Deployen der SSH-Keys für Nodes 2-4
 
2. Playbook mit Erklärung der einzelnen Zeilen (absent, exclusive, ...)

    ```
    - name: Public key is deployed to managed hosts for Ansible
      hosts: all

      tasks:
      - name: Ensure key is in root's ~/.ssh/authorized_hosts
        authorized_key:
         user: root
         state: present
         key: '{{ item }}'
        with_file:
         - ~/.ssh/id_rsa.pub
    ```

3. Befehl zum Ausführen des Playbooks:

    ```
    ansible-playbook <playbook-name>
    ```


4. Fehler andeuten, und Erklärung, da Authentifizierung (Erstkontakt) nicht stattfinden kann ohne PW:


```
ansible-playbook <playbook-name> --ask-pass
```

Alternativ:
Alternativ kann man in der Inventory angeben, welchen User wir verwenden und welches Passwort. Das Passwort sollte normalerweise nicht 
als Klartext eingefügt werden. Hierfür gibt es Ansible Vault, womit Passwörter verschlüsselt werden können. 
Group_vars oder host_vars und dann mit ansible vault verschlüsseln

```
$ itsserver1 ansible_host=10.0.0.11 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
$ itsserver2 ansible_host=10.0.0.12 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin 
$ itsserver3 ansible_host=10.0.0.13 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
$ itsserver4 ansible_host=10.0.0.14 ansible_ssh_user=itsadmin ansible_ssh_pass=itsadmin
```

5. Unterschied von manueller Verteilung für ein Host und Verteilung mit einem Playbook für mehrere Hosts

6. Als Rolle hinzufügen (Best Practice: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

    1. Roles Ordner erstellen
    2. Neuer Ordner mit Rollen-Name => deploy_ssh_keys
    3. Ordner "tasks" erstellen
    4. main.yml erstellen
    5. Code hinzufügen von Punkt 2



# User erstellen mit Variablen und Passwörter in vars datei mit Ansible Vault verschlüsseln

Wir erstellen nun eine neue Rolle "add user" um einen User zu erstellen.

```
$ cd roles
$ mkdir add_user
$ mkdir tasks
$ vim add_user
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

Nun können wir zum Beispiel eine Zeile hinzufügen, wenn wir zum Beispiel einen User zu adm hinzufügen möchten, um nicht ständig als
root arbeiten müssen.

```
- name: Add user
  hosts: all
  tasks:
    - name: Add a new user with username, commentname and group
      user:
        name: "{{ username }}"
        comment: "{{ name }}"
        group: "{{ groupname }}"
```

Testen wir dies einmal mit folgendem Befehl:


```
ansible-playbook main.yml -e "username=test name=test groupname=add"
```

# Software Docker installieren und Container automatisiert erstellen

# Rolle für Webserver erstellen mit LAMP

Test am Ende:
localhost um apache zu testen

phpinfo.php in /var/www/html/ erstellen mit folgendem Inhalt:

<?php
phpinfo();
?>

localhost/phpinfo.php aufrufen

# GitLab 

# Playbook mit allen Roles

## yum Modul hinzufügen

Wir fügen als erstes das apt Modul hinzu, um zu aller erst alle Pakete zu prüfen und ggf. upzudaten.

Wir erstellen hierfür in unserem Projekt eine Datei `main.yml` und fügen folgendes hinzu:

```
- name: Main Playbook to configure every server with one playbook to all hosts
  hosts: all
  tasks:
    - name: Upgrade all packages
      apt:
        name: '*'
        state: latest
```

## Rollen einfügen und alles ausführen

# Ansible Vault

# Ansible AWX

# Ansible Galaxy

# Ansible Doc