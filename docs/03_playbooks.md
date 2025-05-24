# 03 – Playbooks Ansible pour le déploiement de GLPI

---

## Playbook : `install-glpi-deps.yml`

### Objectif

Ce playbook installe les packages nécessaires au fonctionnement de GLPI, notamment :
- Apache2 (serveur web)
- MariaDB (serveur de base de données)
- PHP 8.2 et les extensions requises : `cli`, `curl`, `gd`, `intl`, `ldap`, `mbstring`, `mysql`, `xml`, `zip`
- Le module `python3-pymysql` requis par Ansible pour interagir avec MariaDB

Il redémarre ensuite Apache2 pour appliquer les modifications.

> **Remarque** : tous les playbooks sont regroupés dans un dossier dédié `glpi-ansible-debian12/` dans le home de l’utilisateur `ansible`.

> Le fichier [`/ansible/hosts`](../inventory/hosts) doit obligatoirement spécifier l’utilisateur `ansible` avec la directive `ansible_user=ansible` pour que l’exécution soit fonctionnelle.

---

### Fichier source

> Le fichier source est disponible ici : [`install-glpi-deps.yml`](../playbooks/install-glpi-deps.yml)

---

### Aperçu du playbook

![Playbook install-glpi-deps.yml](/captures/playbook_install_glpi.png)

---

### Résultat de l’exécution

![Résultat d’exécution](/captures/playbook_install_glpi_ok.png)

---

##  Playbook `mariadb-config.yml`  

### Objectif

Ce playbook configure le serveur de base de données MariaDB pour l’application GLPI.  
Il effectue les actions suivantes :

- Définit un mot de passe root (via socket Unix pour élévation automatique)
- Crée la base `glpi`
- Crée l’utilisateur `mbits` avec son mdp
- Accorde les privilèges locaux et distants à cet utilisateur pour la base `glpi`

---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/mariadb-config.yml`](../playbooks/mariadb-config.yml)

---

### Aperçu du playbook

![playbook_install_mariadb_1](/captures/playbook_install_mariadb_1.png)  
![playbook_install_mariadb_2](/captures/playbook_install_mariadb_2.png)

---

### Résultat de l’exécution

![playbook_install_mariadb_okpng](/captures/playbook_install_mariadb_okpng.png)

---

## Playbook `download-glpi.yml`

### Objectif

Ce playbook télécharge et décompresse l’archive officielle de GLPI dans le répertoire `/var/www/html`, puis applique les droits nécessaires au bon fonctionnement du service web.

Les tâches effectuées sont :

- Création d’un dossier temporaire `/tmp/glpi`
- Téléchargement de l’archive `.tgz` de GLPI depuis GitHub
- Extraction dans `/var/www/html`
- Suppression de l’archive après décompression
- Application des droits `www-data:www-data` récursivement sur `/var/www/html/glpi`

---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/download-glpi.yml`](../playbooks/download-glpi.yml)

---

### Aperçu du playbook

![playbook_download_glpi](/captures/playbook_download_glpi.png)

---

### Résultat de l’exécution

![playbook_download_glpi_ok](/captures/playbook_download_glpi_ok.png)

---

