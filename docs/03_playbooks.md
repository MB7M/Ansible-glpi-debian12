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

## Playbook `install-glpi-cli.yml`

### Objectif

Ce playbook permet de réaliser l'installation silencieuse de GLPI via la ligne de commande (`bin/console`)  
Il vérifie d'abord la connexion à la base de données MariaDB, puis exécute l'installation CLI.

Les variables de connexion à la base sont définies au début du fichier pour faciliter la gestion et la lisibilité.

### Fichier source

> Le fichier source est disponible ici : [`playbooks/install-glpi-cli.yml`](../playbooks/install-glpi-cli.yml)

---

### Aperçu du playbook

![playbook_install_glpi_cli_1](/captures/playbook_install_glpi_cli_all.png)  

---

### Résultat de l’exécution

![playbook_install_glpi_cli_ok](/captures/playbook_install_glpi_cli_ok.png)

---

## Playbook `glpi-ssl.yml`

### Objectif

Ce playbook génère un certificat SSL auto-signé et configure Apache2 pour sécuriser l’accès à GLPI via HTTPS.  
Il effectue les opérations suivantes :

- Création des répertoires `/etc/ssl/certs` et `/etc/ssl/private`
- Génération d’une clé privée et d’un CSR (Certificate Signing Request)
- Génération d’un certificat SSL auto-signé
- Installation de la librairie Python requise (`python3-cryptography`)
- Copie d’un fichier de configuration Apache (`glpi-ssl.conf`) préparé en amont
- Activation du site via `a2ensite` et redémarrage d’Apache

> ⚠️ Le fichier `/home/ansible/glpi-ssl.conf` **doit être préparé manuellement** avec les bons chemins vers les certificats avant exécution du playbook.

>  Assurez-vous que le service Apache2 est actif et opérationnel avant l’exécution.

> L’accès HTTPS ne sera pleinement fonctionnel qu’après avoir :
> - ouvert GLPI via navigateur depuis la machine cliente
> - accepté manuellement le certificat (auto-signé)
> - ajouté l’URL avec le CN utilisé (`glpi.mbits.lan`) dans les hôtes de confiance (par exemple dans les certificats autorisés sous Windows)

---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/glpi-ssl.yml`](../playbooks/glpi-ssl.yml)

---

### Aperçu du playbook

![playbook_glpi_ssl_1](/captures/playbook_glpi_ssl_1.png)  
![playbook_glpi_ssl_2](/captures/playbook_glpi_ssl_2.png)

---

### Résultat de l’exécution

![playbook_glpi_ssl_ok](/captures/playbook_glpi_ssl_ok.png)

---

### Vérification côté navigateur

![certificas_ssl](/captures/certificas_ssl.png)  
![glpi_https_ok](/captures/glpi_https_ok.png)


