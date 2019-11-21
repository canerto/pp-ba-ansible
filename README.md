# Ansible

## Grundlagen

### Control Node

### Managed Nodes

### Configuration Files

Die Konfigurationsdatei `/etc/ansible/ansible.cfg` wird nach der Installation von Ansible angelegt. Diese primäre Datei beinhaltet
Default-Einstellungen, z.B. der Pfad zur Inventory.

Eine Beispieldatei hierfür ist verfügbar in GitHub unter https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg und weitere Informationen
sind verfügbar unter dem offiziellen Ansible-Link https://docs.ansible.com/ansible/latest/cli/ansible-config.html#ansible-config


### Inventory

Das Inventory File, meistens auch "Hostfile" genannt, liegt unter `/etc/ansible/hosts`. Der Pfad hierzu kann in der Konfigurationsdatei ansible.cfg geändert werden. 
In dieser Datei wird eine Liste definiert, in der die zu verwaltenden Hosts von Ansible aufgelistet werden.

### Modules

### Ad-hoc Tasks

### Playbooks

### Variables

### Facts

### Roles



## Installation Control Node unter Debian

Um die aktuellste Version von Ansible für Ubuntu zu nutzen, kann man die PPA (personal package archive) dem System hinzufügen.

Dafür benötigt man zuerst das `software-properties-common` Paket, um Software Repos einfacher zu managen. 
Anschließend das Ansible PPA hinzufügen und dann Ansible installieren.

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

 


# Ansible AWX

# Ansible Vault