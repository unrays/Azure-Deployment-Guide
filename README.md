# Documentation
Ce document présente les différentes étapes de configuration d’un serveur Azure, depuis la mise en place de la connexion SSH jusqu’à la sécurisation avancée et l’hébergement de plusieurs sites web. Chaque section détaille les commandes à exécuter et les configurations à réaliser.

---

## 1. Configuration SSH

Pour établir une connexion sécurisée avec le serveur Azure, il est essentiel d’utiliser SSH.

### 1.1. Préparation sur la machine cliente

- **Localisation du fichier de configuration :**  
  Rendez-vous dans le dossier `.ssh` de l’utilisateur courant sur Windows.

- **Ajout de la clé RSA :**  
  Placez votre clé RSA obtenue sur Azure dans ce dossier.

- **Modification du fichier de configuration :**  
  Ajoutez les lignes suivantes pour lier votre clé à l’adresse IP publique du serveur Azure :

  ```ssh
  Host {alias ssh}
     HostName {adresse IP}
     User {username}
     IdentityFile {Emplacement de la clé}
  ```

### 1.2. Connexion au serveur

Pour vous connecter via SSH, lancez simplement :

```bash
ssh {alias ssh}
```

*Sortie attendue :*
```bash
{username}@{servername}:~$
```

---

## 2. Configuration du Pare-feu (UFW)

La sécurisation du serveur passe par la configuration d’un pare-feu.

### 2.1. Installation et réinitialisation

Exécutez les commandes suivantes :

1. **Installation de UFW :**

   ```bash
   sudo apt-get install ufw
   ```

2. **Réinitialisation de la configuration :**

   ```bash
   sudo ufw reset
   ```

### 2.2. Ouverture des ports

Ouvrez les ports nécessaires pour le fonctionnement du serveur :

```bash
sudo ufw allow 22/tcp   # Port SSH
sudo ufw allow 80/tcp   # Port HTTP
sudo ufw allow 443/tcp  # Port HTTPS
```

### 2.3. Activation et vérification

Activez le pare-feu et vérifiez son statut :

```bash
sudo ufw enable
sudo ufw status
```

---

## 3. Installation et Configuration de Fail2ban

Fail2ban protège efficacement le serveur contre les attaques par force brute en analysant les logs de connexion.

### 3.1. Installation et configuration

1. **Installation de Fail2ban :**

   ```bash
   sudo apt-get install fail2ban
   ```

2. **Création ou modification du fichier de configuration :**

   ```bash
   sudo [micro/nano/vim] /etc/fail2ban/jail.local
   ```

3. **Redémarrage et vérification du service :**

   ```bash
   sudo systemctl restart fail2ban
   sudo systemctl status fail2ban
   sudo fail2ban-client status
   ```

---

## 4. Mise en Place du Serveur Web

Cette section regroupe l’installation et la configuration d’Apache, PHP, MySQL et PHPMyAdmin.

### 4.1. Installation d’Apache

1. **Installation d’Apache :**

   ```bash
   sudo apt install apache2
   ```

2. **Vérification de la page par défaut :**

   ```bash
   sudo [micro/nano/vim] /var/www/html/index.html
   ```

3. **Test dans le navigateur :**  
   Rendez-vous sur [http://adresse.ip.du.serveur/](http://adresse.ip.du.serveur/)

> **Note :** Pour une configuration sur Azure, pensez à ajouter une règle de trafic entrant pour le port 80 via l’interface web du serveur.

### 4.2. Installation et Configuration de PHP

1. **Installation de PHP :**

   ```bash
   sudo apt install php libapache2-mod-php
   ```

2. **Vérification de la version de PHP :**

   ```bash
   php -v | grep --only-matching --perl-regexp "(PHP )\d+\.\d+\.\d+" | cut -c 5-7
   ```

3. **Activation et désactivation des modules Apache :**

   ```bash
   sudo a2enmod php8.3
   sudo a2dismod php8.3
   ```

### 4.3. Installation et Configuration de MySQL

1. **Installation de MySQL Server :**

   ```bash
   sudo apt install mysql-server
   ```

2. **Configuration initiale de MySQL :**

   ```bash
   sudo mysql_secure_installation
   ```

   Lors de cette configuration, suivez les étapes :

   - Entrer le mot de passe
   - Pour "VALIDATE PASSWORD component" : **N**
   - Pour "Change password for root" : **N**
   - Pour "Remove anonymous users" : **Y**
   - Pour "Disallow root login remotely" : **Y**
   - Pour "Remove test database and access to it" : **Y**
   - Pour "Reload privileges table now" : **Y**

3. **Création de l’utilisateur root :**

   ```bash
   sudo mysql --u root
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin1! ';
   flush privileges;
   exit;
   ```

4. **Création de l’utilisateur « superadmin » :**

   ```bash
   sudo mysql --u root --p
   CREATE USER 'superadmin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin1!';
   GRANT ALL PRIVILEGES ON *.* TO 'superadmin'@'localhost';
   GRANT GRANT OPTION ON *.* TO 'superadmin'@'localhost';
   flush privileges;
   exit;
   ```

### 4.4. Installation et Configuration du Module PHP-MySQL

1. **Installation du module :**

   ```bash
   sudo apt install php-mysql
   ```

2. **Configuration des modules et affichage des erreurs :**

   Ouvrez le fichier de configuration PHP :

   ```bash
   sudo [micro/nano/vim] /etc/php/8.3/apache2/php.ini
   ```

   Ajoutez les lignes suivantes à la fin du fichier :

   ```ini
   [MySQL]
   extension = mysqli
   extension = pdo_mysql
   display_errors = On
   display_startup_errors = On
   opcache.enable = 0
   log_errors = On
   mysqlnd.debug = /var/log/php-mysql
   ```

3. **Redémarrage des services :**

   ```bash
   sudo systemctl restart apache2
   sudo systemctl restart mysql
   ```

### 4.5. Installation et Configuration de PHPMyAdmin

1. **Installation de PHPMyAdmin :**

   ```bash
   sudo apt install phpmyadmin
   ```

2. **Configuration pendant l’installation :**

   - Sélectionnez **apache2** (barre d’espace puis tab pour « Ok »).
   - Sélectionnez **yes**.
   - Laissez le mot de passe vide.

3. **Test de PHPMyAdmin :**  
   Rendez-vous sur [http://adresse.ip.du.serveur/phpmyadmin](http://adresse.ip.du.serveur/phpmyadmin) et connectez-vous avec l’utilisateur **superadmin**.

---

## 5. Configuration des VHOST pour l’Hébergement de Sites Web

Pour héberger plusieurs sites, nous allons créer des Virtual Hosts (VHOST) avec Apache.

### 5.1. Création des Répertoires et Attribution des Droits

1. **Création des répertoires pour les sites :**

   ```bash
   sudo mkdir {blog/wiki/www}.{adresse IP}.nip.io
   ```

2. **Attribution des droits au serveur web :**

   ```bash
   sudo chown {votre username}:www-data -R {blog/wiki/www}.{adresse IP}.nip.io
   sudo chmod 0755 {blog/wiki/www}.{adresse IP}.nip.io
   ```

3. **Création des fichiers HTML de base :**

   ```bash
   cd {blog/wiki/www}.{adresse IP}.nip.io
   sudo [micro/nano/vim] index.php
   ```

   Collez le contenu suivant (en remplaçant « unknown » par votre nom) :

   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>W3.CSS Template</title>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1">
     <link rel="stylesheet" href="https://www.w3schools.com/w3css/5/w3.css">
     <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Raleway">
     <style>
       body,h1,h2,h3,h4,h5 {font-family: "Raleway", sans-serif}
     </style>
   </head>
   <body class="w3-light-grey">
   ```

### 5.2. Création et Activation du Fichier de Configuration VHOST

1. **Création du fichier de configuration :**

   ```bash
   cd /etc/apache2/sites-available
   sudo [micro/nano/vim] {blog/wiki/www}.{adresse IP}.nip.io.conf
   ```

2. **Collez la configuration suivante :**

   ```apacheconf
   <VirtualHost *:80>
       ServerAdmin {votre email pour l'administration}
       ServerName {blog/wiki/www}.{adresse IP}.nip.io
       DocumentRoot /var/www/{blog/wiki/www}.{adresse IP}.nip.io/
       ErrorLog ${APACHE_LOG_DIR}/{blog/wiki/www}.{adresse IP}.nip.io.error.log
       CustomLog ${APACHE_LOG_DIR}/{blog/wiki/www}.{adresse IP}.nip.io.access.log combined
   </VirtualHost>
   ```

3. **Activation du VHOST et rechargement d’Apache :**

   ```bash
   sudo a2ensite {blog/wiki/www}.{adresse IP}.nip.io.conf
   sudo systemctl reload apache2
   sudo chmod 0755 /var/log/apache2/
   ls -l /var/log/apache2
   ```

> **Test :** Vérifiez le fonctionnement du site dans votre navigateur.

---

## 6. Installation et Configuration des Applications Web

Trois sites seront mis en place : un blog, un wiki et un site basé sur FlatPress (picoCMS).

### 6.1. Configuration de FlatPress

1. **Installation de `unzip` :**

   ```bash
   sudo apt-get install unzip
   ```

2. **Téléchargement et installation de FlatPress :**

   ```bash
   cd /var/www
   sudo wget https://github.com/flatpressblog/flatpress/archive/refs/tags/1.3.1.zip
   sudo mv 1.3.1.zip flatpress.zip
   sudo unzip flatpress.zip
   sudo mv flatpress-1.3.1/* /var/www/blog.{adresse IP}.nip.io/
   sudo chown -R www-data:www-data /var/www/blog.{adresse IP}.nip.io/
   sudo chmod g+w /var/www/blog.{adresse IP}.nip.io/
   ```

> Rendez-vous sur le site pour compléter la configuration.

### 6.2. Configuration de DokuWiki

1. **Téléchargement de DokuWiki :**

   ```bash
   cd /var/www
   sudo wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
   ```

2. **Extraction et déplacement des fichiers :**

   ```bash
   sudo tar -xvzf dokuwiki-stable.tgz
   sudo mv /var/www/dokuwiki-2024-02-06b/* /var/www/wiki.{adresse IP}.nip.io/
   sudo chown -R www-data:www-data /var/www/wiki.{adresse IP}.nip.io/
   ```

### 6.3. Configuration de PicoCMS

1. **Installation de Composer et téléchargement de PicoCMS :**

   ```bash
   cd /home/{nom d'utilisateur}
   curl -sSL https://getcomposer.org/installer | php
   php composer.phar create-project picocms/pico-composer pico
   ```

2. **Déplacement de PicoCMS dans le répertoire Apache approprié :**

   ```bash
   sudo mv pico/* /var/www/www.{adresse IP}.nip.io/
   ```

---

## 7. Sécurisation SSL et Redirection HTTP vers HTTPS

Pour sécuriser les échanges, il est nécessaire de créer un certificat SSL et de rediriger les requêtes HTTP vers HTTPS.

### 7.1. Installation des outils nécessaires

1. **Activation du module SSL d’Apache :**

   ```bash
   sudo a2enmod ssl
   sudo systemctl restart apache2
   ```

2. **Installation de `certutil` et `mkcert` :**

   ```bash
   sudo apt install libnss3-tools
   curl -JLO https://dl.filippo.io/mkcert/latest?for=linux/amd64
   chmod +x mkcert-v*-linux-amd64
   sudo cp mkcert-v*-linux-amd64 /usr/local/bin/mkcert
   ```

### 7.2. Création du Certificat

1. **Installation du CA :**

   ```bash
   mkcert --install
   ```

2. **Création du certificat :**

   ```bash
   mkcert certificat  # ou le nom que vous souhaitez
   ```

### 7.3. Configuration des VHOST SSL

1. **Création d’un nouveau fichier VHOST pour rediriger HTTP vers HTTPS :**

   ```bash
   cd /etc/apache2/sites-available
   sudo [micro/nano/vim] {blog/wiki/www}-ssl.conf
   ```

2. **Collez le contenu suivant :**

   ```apacheconf
   <VirtualHost *:80>
       ServerName {blog/wiki/www}.{adresse IP}.nip.io
       DocumentRoot /var/www/{blog/wiki/www}.{adresse IP}.nip.io/
       Redirect / https://{blog/wiki/www}.{adresse IP}.nip.io/
   </VirtualHost>
   ```

3. **Activation du VHOST SSL :**

   ```bash
   sudo a2ensite {blog/wiki/www}-ssl.conf
   ```

> Répétez cette opération pour chacun des sites concernés.

---

## 8. Gestion des Utilisateurs et des Groupes

Afin de compartimenter les accès aux différents services, plusieurs utilisateurs et groupes sont créés.

### 8.1. Création des Utilisateurs pour les Sites

1. **Création d’un utilisateur pour chaque site (blog, wiki, www) :**

   ```bash
   sudo adduser programmeur_{blog/wiki/www}
   ```

   - Entrez le mot de passe (aucun dans ce cas) et ignorez les autres étapes.

2. **Attribution de la propriété du site :**

   ```bash
   sudo chown -R programmeur_{blog/wiki/www} /var/www/blog.{adresse IP}.nip.io
   ```

### 8.2. Création d’un Utilisateur Master et d’un Groupe Global

1. **Création de l’utilisateur master :**

   ```bash
   sudo adduser programmeur_master
   ```

2. **Création du groupe et ajout des utilisateurs :**

   ```bash
   sudo groupadd programmeurs_globaux
   sudo usermod -a -G programmeurs_globaux programmeur_master
   sudo usermod -a -G programmeurs_globaux www-data
   ```

3. **Attribution des permissions au groupe sur les sites :**

   ```bash
   sudo chown -R :programmeurs_globaux /var/www/{blog/wiki/www}.{adresse IP}.nip.io
   sudo chmod g+rwx /var/www/{blog/wiki/www}.{adresse IP}.nip.io
   ```

---

## 9. Script de Sécurité SFTP

Pour limiter l’accès au shell et sécuriser les connexions SFTP, un script spécifique est créé.

### 9.1. Création du Script

1. **Création du fichier de script :**

   ```bash
   sudo [micro/nano/vim] /usr/local/bin/sftponly
   ```

2. **Collez le contenu suivant dans le fichier :**

   ```bash
   #!/bin/bash
   if [[ "$2" = *sftp-server ]]
   then
       exec /bin/bash "$@"
   else
       echo "'$LOGNAME' n'a pas accès au shell"
       exit 1
   fi
   ```

3. **Ajout des droits d’exécution :**

   ```bash
   sudo chmod +x /usr/local/bin/sftponly
   ```

4. **Modification du shell d’un utilisateur pour utiliser le script :**

   ```bash
   sudo usermod --shell /usr/local/bin/sftponly {nom d'utilisateur}
   ```

> **Attention :** Cette manipulation peut compromettre l’accès au serveur si la syntaxe est incorrecte ou si vous modifiez l’utilisateur par défaut.

---

## 10. Génération d’un Accès SSH pour Chaque Utilisateur

Pour chaque utilisateur créé, il faut générer une paire de clés SSH et configurer l’accès.

### 10.1. Génération de la Clé SSH

Sur la machine client, exécutez :

```bash
ssh-keygen -t ed25519 -C "{nom d'utilisateur}@azure"
```

- Entrez un nom de fichier (souvent celui de l’utilisateur).
- Laissez vide les autres champs.

### 10.2. Copie de la Clé Publique

Pour afficher et copier la clé générée :

```bash
type {nom du fichier}.pub
```

L’output ressemblera à :

```bash
ssh-ed25519 {cle} {utilisateur@azure}
```

### 10.3. Configuration du Fichier `config` du Client SSH

Ajoutez les lignes suivantes dans le fichier `config` situé dans le dossier `.ssh` de votre utilisateur Windows :

```ssh
Host {nom de l'utilisateur}_tp2
    HostName 4.248.185.45
    User {nom de l'utilisateur}
    IdentityFile "C:\Users\etudiant\{nom du fichier}"
```

### 10.4. Configuration sur le Serveur pour Chaque Utilisateur

1. **Connexion en tant qu’administrateur puis passage à l’utilisateur concerné :**

   ```bash
   sudo -i -u {nom d'utilisateur}
   ```

2. **Création du dossier `.ssh` et du fichier `authorized_keys` :**

   ```bash
   mkdir .ssh
   cd .ssh
   {micro/nano/vim} authorized_keys
   ```

   Collez-y la clé publique générée.

3. **Redémarrage du service SSH :**

   ```bash
   exit
   sudo systemctl restart ssh
   ```

> Testez ensuite la connexion SSH avec l’utilisateur concerné.

Version définitive mise en page au format Markdown par ChatGPT le dimanche 13 avril 2025.

---

## License

**This project is licensed under the [MIT License](LICENSE).**  
You are free to use, modify, and distribute this work, provided that the license terms are respected.

&copy; 2025 **Félix-Olivier Dumas**  
_All rights reserved unless otherwise stated in the license file._

---
