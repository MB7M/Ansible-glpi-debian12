# 03 – Playbooks Ansible pour le déploiement de GLPI

---

## 01. Playbook : `install-glpi-deps.yml`

### Objectif

Ce playbook installe les packages nécessaires au fonctionnement de GLPI, notamment :
- Apache2 (serveur web)
- MariaDB (serveur de base de données)
- PHP 8.2 et les extensions requises : `cli`, `curl`, `gd`, `intl`, `ldap`, `mbstring`, `mysql`, `xml`, `zip`
- Le module `python3-pymysql` requis par Ansible pour interagir avec MariaDB

Il redémarre ensuite Apache2 pour appliquer les modifications.

> **Remarque** : tous les playbooks sont regroupés dans un dossier dédié `glpi-ansible-debian12/` dans le home de l’utilisateur `ansible`.

> Le fichier [`etc/ansible/hosts`](../inventory/hosts) doit obligatoirement spécifier l’utilisateur `ansible` avec la directive `ansible_user=ansible` pour que l’exécution soit fonctionnelle.

La commande d’exécution à utiliser est la suivante (en adaptant le nom du fichier selon le playbook concerné) :

```bash
ansible-playbook nomduplaybook.yml -u ansible (utilisateur)
```
---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/01_install-glpi-deps.yml`](../playbooks/01_install-glpi-deps.yml)

---

### Aperçu du playbook

![Playbook install-glpi-deps.yml](/captures/playbook_install_glpi.png)

---

### Résultat de l’exécution

![Résultat d’exécution](/captures/playbook_install_glpi_ok.png)

---

## 02. Playbook `mariadb-config.yml`  

### Objectif

Ce playbook configure le serveur de base de données MariaDB pour l’application GLPI.  
Il effectue les actions suivantes :

- Définit un mot de passe root (via socket Unix pour élévation automatique)
- Crée la base `glpi`
- Crée l’utilisateur `mbits` avec son mdp
- Accorde les privilèges locaux et distants à cet utilisateur pour la base `glpi`

---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/02_mariadb-config.yml`](../playbooks/02_mariadb-config.yml)

---

### Aperçu du playbook

![playbook_install_mariadb_1](/captures/playbook_install_mariadb_1.png)  
![playbook_install_mariadb_2](/captures/playbook_install_mariadb_2.png)

---

### Résultat de l’exécution

![playbook_install_mariadb_okpng](/captures/playbook_install_mariadb_okpng.png)

---

## 03. Playbook `download-glpi.yml`

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

> Le fichier source est disponible ici : [`playbooks/03_download-glpi.yml`](../playbooks/03_download-glpi.yml)

---

### Aperçu du playbook

![playbook_download_glpi](/captures/playbook_download_glpi.png)

---

### Résultat de l’exécution

![playbook_download_glpi_ok](/captures/playbook_download_glpi_ok.png)

---

## 04. Playbook `install-glpi-cli.yml`

### Objectif

Ce playbook permet de réaliser l'installation silencieuse de GLPI via la ligne de commande (`bin/console`)  
Il vérifie d'abord la connexion à la base de données MariaDB, puis exécute l'installation CLI.

Les variables de connexion à la base sont définies au début du fichier pour faciliter la gestion et la lisibilité.

### Fichier source

> Le fichier source est disponible ici : [`playbooks/04_install-glpi-cli.yml`](../playbooks/04_install-glpi-cli.yml)

---

### Aperçu du playbook

![playbook_install_glpi_cli_1](/captures/playbook_install_glpi_cli_all.png)  

---

### Résultat de l’exécution

![playbook_install_glpi_cli_ok](/captures/playbook_install_glpi_cli_ok.png)

---

## 05. Playbook `glpi-ssl.yml`

### Objectif

Ce playbook génère un certificat SSL auto-signé et configure Apache2 pour sécuriser l’accès à GLPI via HTTPS.  
Il effectue les opérations suivantes :

- Création des répertoires `/etc/ssl/certs` et `/etc/ssl/private`
- Génération d’une clé privée et d’un CSR (Certificate Signing Request)
- Génération d’un certificat SSL auto-signé
- Installation de la librairie Python requise (`python3-cryptography`)
- Copie d’un fichier de configuration Apache (`glpi-ssl.conf`) préparé en amont
- Activation du site via `a2ensite` et redémarrage d’Apache

> ⚠️ Le fichier `/home/ansible/glpi-ssl.conf` **doit être préparé manuellement** avec les bons chemins vers les certificats avant exécution du playbook :

```apache
<VirtualHost *:443>
    ServerName glpi.mbits.lan
    DocumentRoot /var/www/html/glpi

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/certificat_glpi.crt
    SSLCertificateKeyFile /etc/ssl/private/cle_glpi.key

    <Directory /var/www/html/glpi>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
</VirtualHost>
```

>   Ce playbook suppose que le module Apache ssl  a déjà été activé manuellement. Si ce n’est pas le cas, exécute la commande suivante sur le serveur GLPI avant lancement :

```bash
a2enmod ssl && systemctl restart apache2
```
> L’accès HTTPS ne sera pleinement fonctionnel qu’après avoir :
> - ouvert GLPI via navigateur depuis la machine cliente
> - accepté manuellement le certificat (auto-signé)
> - ajouté l’URL avec le CN utilisé (`glpi.mbits.lan`) dans les hôtes de confiance (par exemple dans les certificats autorisés sous Windows)

---

### Fichier source

> Le fichier source est disponible ici : [`playbooks/05_glpi-ssl.yml`](../playbooks/05_glpi-ssl.yml)

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


