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

Wir haben nun die Hosts angegeben, mit der IP-Adresse und welchen User wir verwenden. Das Passwort sollte normalerweise nicht als Klartext
eingefügt werden. Hierfür gibt es Ansible Vault, womit Passwörter verschlüsselt werden können. Dazu kommen wir aber später.

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

### SSH-Konfiguration

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


### SSH-Key-Verteilung 

## explizit für einen Node
Der public Key muss nun auf die zu verwaltenden Nodes kopiert werden.

```
$ cd /root/.ssh/
$ ssh-copyid root@<entfernter Node>
```

Im Anschluss versuchen wir eine SSH-Verbindung auf den zu verwaltenden Node aufzubauen, ohne ein Passwort eingeben zu müssen.

```
$ ssh root@<entfernter Node>
```

## mit einem Playbook für alle Nodes

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

4. Fehler andeuten und Erklärung, da Authentifizierung (Erstkontakt) nicht stattfinden kann ohne PW:

```
ansible-playbook <playbook-name> --ask-pass
```

5. Unterschied von manueller Verteilung für ein Host und Verteilung mit einem Playbook für mehrere Hosts

### Weniger priviligierte Accounts erstellen und Gruppen zuordnen

### Software und OS updaten

### Software und OS automatisiert updaten (zeitgesteuerte Jobs)

### Software Docker installieren und Container automatisiert erstellen

### GitLab 

# Ansible Vault

# Ansible AWX

# Ansible Galaxy

# Ansible Doc