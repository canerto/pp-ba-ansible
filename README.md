# Ansible

## Grundlagen

### Control Node

Man spricht von Control Nodes, wenn Ansible auf einer Maschine installiert ist. Hierfür kann man jeden Computer nutzen, auf dem Python installiert ist.
Von hier aus kann man Playbooks und Aufgaben laufen lassen. Außerdem kann man mehrere Control Nodes haben, jedoch kann man kein Computer mit dem Windowsbetriebssystem als Control Node nutzen.

### Managed Nodes

Managed Nodes, auch "Hosts" genannt, sind die Geräte, die verwaltet werden.

### Configuration Files

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

### SSH-Konfiguration

Ansible kommuniziert primär mit den Managed Nodes via SSH. Zuerst erzeugen wir ein SSH-Schlüsselpaar auf dem Control Node,
um später eine passwortlose Anmeldung auf den Managed Nodes zu ermöglichen.

```
$ ssh-keygen
```

Die folgenden Fragen können wir mit Return bestätigen. Dadurch entstehen nun im /.ssh/ Verzeichnis zwei Dateien. Es handelt sich um 
den private und public Key.
Der public Key muss nun auf die zu verwaltenden Nodes kopiert werden.

```
$ cd /root/.ssh/
$ ssh-copyid root@<entfernter Node>
```

Im Anschluss versuchen wir eine SSH-Verbindung auf den zu verwaltenden Node aufzubauen, ohne ein Passwort eingeben zu müssen.

```
$ ssh root@<entfernter Node>
```


# Ansible AWX

# Ansible Vault