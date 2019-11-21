# Ansible

## Grundlagen

### Control Node

### Managed Nodes

### Inventory

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