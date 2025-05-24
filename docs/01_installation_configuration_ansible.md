# 01 – Installation et configuration d’Ansible

## Objectif

Mettre en place Ansible sur la machine SRV-ANSIBLE (Debian 12), configurer son inventaire et valider la connexion à la machine distante (SRV-GLPI).

---

## A) Installation des prérequis et d’Ansible

Cette méthode repose sur l'ajout du dépôt officiel recommandé dans la documentation Ansible. Elle permet d’installer une version récente, sans utiliser `apt-key` obsolète niveau sécurité.

### Étape 1 – Définir la version Ubuntu compatible

```bash
export UBUNTU_CODENAME=jammy
```
> Debian 12 est compatible avec Ubuntu 22.04 (Jammy), utilisé dans les dépôts officiels.

---

### Étape 2 – Installer les outils nécessaires

```bash
sudo apt update && sudo apt install -y wget gpg
```

- `wget` : téléchargement de fichiers
    
- `gpg` : gestion des clés pour dépôt signé
    

---

### Étape 3 – Télécharger et enregistrer la clé GPG officielle d’Ansible

```bash
wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" \ | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
```

Cette clé permet à `apt` de vérifier que le dépôt est bien signé par Ansible.

---

### Étape 4 – Ajouter le dépôt sécurisé

```bash
echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" \ | sudo tee /etc/apt/sources.list.d/ansible.list
```

---

### Étape 5 – Installer Ansible et les dépendances

```bash
sudo apt update
sudo apt install -y ansible python3-cryptography python3-pymysql
```

- `python3-cryptography` : nécessaire pour le chiffrement (notamment lors des connexions SSH sécurisées ou modules Ansible utilisant SSL).  
 - `python3-pymysql` : permet à Ansible de gérer MySQL/MariaDB via le module `mysql_*`.
---

### Étape 6 – Vérifier la version

```bash
ansible --version
```
![captures](/captures/ansible_version.png)

---

## B) Configuration d’Ansible

---

### Étape 1 – Générer un fichier de configuration complet

Créer le dossier de configuration s’il n’existe pas, il est nécessaire pour centraliser la configuration d’Ansible (`ansible.cfg`) et l’inventaire (`hosts`) sur Debian. Il n’est pas créé automatiquement à l’installation.

```bash
sudo mkdir -p /etc/ansible 
```
ensuite, générer le fichier de configuration Ansible
```bash
sudo ansible-config init --disabled -t all > /etc/ansible/ansible.cfg
```

> Le fichier `/etc/ansible/ansible.cfg` contient toutes les options commentées. Cela permet de personnaliser finement la configuration si besoin.

---

### Étape 2 – Créer le fichier d’inventaire

```bash
sudo nano /etc/ansible/hosts
```
Contenu :
```ini
[glpi] 172.168.1.47 ansible_user=wilder
```

![captures](/captures/inventaire_hosts_glpi.png)

>Le groupe `[glpi]` cible le serveur distant `SRV-GLPI`  avec ici l’utilisateur `wilder` pour les tests initiaux.  
>Cet utilisateur est utilisé uniquement pour vérifier la connexion SSH ; **un utilisateur dédié (`ansible`) sera défini plus tard dans le fichier `hosts` pour l’automatisation.**

---

### Étape 3 – Tester la connexion SSH

```bash
sudo ansible glpi -m ping -u wilder -k
```
- `glpi` : nom du groupe
    
- `-m ping` : vérifie la connectivité SSH
    
- `-u wilder` : utilisateur distant
    
- `-k` : demande de mot de passe SSH
    

⚠️ Le contrôleur Ansible (`SRV-ANSIBLE`) doit **s’être connecté au préalable une première fois en SSH à `wilder@172.168.1.47`**,  
afin que la clé du serveur distant (`SRV-GLPI`) soit enregistrée dans `known_hosts`.  
L’utilisateur `wilder` doit également disposer des droits `sudo` sur la machine cible.

![captures](/captures/ping_noeud_ok.png)
