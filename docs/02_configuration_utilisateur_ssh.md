# 02 – Création de l'utilisateur `ansible` et configuration SSH


## Objectif

Mettre en place un utilisateur dédié (`ansible`) pour l’automatisation sur les deux machines :

- **SRV-ANSIBLE** (maître / contrôleur)

- **SRV-GLPI** (nœud distant)

Cette étape est essentielle pour exécuter les playbooks sans mot de passe, via une authentification par clé SSH.

---

## A) Création de l’utilisateur commun `ansible`

  
### 1. Créer l’utilisateur `ansible` localement sur SRV-ANSIBLE

  
Sur la machine de gestion (SRV-ANSIBLE), l’utilisateur est créé manuellement avec les droits administrateur :


```bash
sudo adduser ansible

sudo usermod -aG sudo ansible
```



### 2. Générer une paire de clés SSH (sur SRV-ANSIBLE)


On génère une clé de type ECDSA pour l’utilisateur `ansible` :


```bash
sudo -u ansible ssh-keygen -t ed25519
```


![ssh_keygen_ansible](/captures/ssh_keygen_ansible.png)

  

> Cette commande génère :

> - `/home/ansible/.ssh/id_ed25519` (clé privée)

> - `/home/ansible/.ssh/id_ed25519.pub` (clé publique)

  

---

### 3. Créer l’utilisateur `ansible` à distance (SRV-GLPI) via Ansible


Sur la machine distante, l’utilisateur `ansible` est créé via le module Ansible `user`, avec les privilèges sudo :


```bash

sudo ansible glpi -m user -a "name=ansible state=present password={{ 'héhé'|password_hash('sha512') }} groups=sudo shell=/bin/bash createhome=yes" -u wilder -b -k -K

```


![creation_utilisateur_ansible](/captures/creation_utilisateur_ansible.png)

  
> Cette commande crée l’utilisateur `ansible` sur le nœud avec :

> - le shell `/bin/bash`

> - les droits sudo

> - un mot de passe chiffré
 
> - options : -k demande le mdp ssh, -K le mdp sudo et -b exécute la tâche avec des privilèges.

  
---

### 4. Vérifier la création du compte à distance

On vérifie avec le module shell :
```bash

sudo ansible glpi -m shell -a "cat /etc/passwd | grep ansible" -u ansible -b -k -K

```


![connexion_ssh_ansible_glpi_ok](/captures/création_shell_ok.png)


> La ligne contenant `ansible:x:` confirme que l’utilisateur est bien créé sur la machine distante.

---

## B) Copie de la clé publique SSH sur le nœud distant

  
On utilise le module `authorized_key` d’Ansible pour copier la clé publique dans le fichier `authorized_keys` de l’utilisateur `ansible` distant.

### 1. Copier la clé publique sur le serveur distant


```bash

sudo ansible glpi -m authorized_key -a "user=ansible state=present key={{ lookup('file','/home/ansible/.ssh/id_ed25519.pub') }}" -u wilder -b -k -K

```

  

![copie_clef_ssh_distant](/captures/copie_clef_ssh_distant.png)

  

> Cette commande injecte la clé publique dans `/home/ansible/.ssh/authorized_keys` sur le nœud GLPI.

  
---

### 2. Vérifier que la clé est bien en place

  
```bash

sudo ansible glpi -m shell -a "cat /home/ansible/.ssh/authorized_keys" -u ansible -b -k -K

```
![copie_clef_ssh_distant](/captures/vérification_shell_clef.png)
  
### 3. Tester la connexion sans mot de passe

```bash
sudo -u ansible ssh ansible@172.168.1.47

```
  
![connexion_ssh_ansible_glpi_ok](/captures/connexion_ssh_ansible_glpi_ok.png)
  

---

## Remarques importantes

  
- L’utilisateur `ansible` doit **appartenir au groupe `sudo`** sur les deux machines.

- L’utilisateur utilisé pour la **copie de la clé (`wilder`)** doit avoir les droits sudo et avoir déjà initié une connexion SSH manuelle pour éviter l’erreur de type `host key verification failed`.
