# Installation

Evolving-HCS est conçu pour fonctionner sur votre propre serveur Web. Vous devrez disposer d'un accès root à votre serveur afin d'exécuter et d'utiliser ce panneau.

Ce panneau n'existe pas en tant que service de glisser-déposer pour faire fonctionner vos serveurs. Il s'agit d'un système très complexe qui nécessite de multiples dépendances et des administrateurs prêts à passer un certain temps à apprendre à l'utiliser. Si vous pensez être capable d'installer ce panneau sans aucune connaissance de l'administration système Linux de base, vous devriez vous arrêter et faire demi-tour maintenant.

## Dependencies
* [PHP `8.0`](https://sys-admin.fr/installation-php-8-0-sur-ubuntu/) with the following extensions: `cli`, `openssl`, `gd`, `mysql`, `PDO`, `mbstring`, `tokenizer`, `bcmath`, `xml` or `dom`, `curl`, `zip`, and `fpm` if you are planning to use nginx
* [MySQL](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04-fr)
* A webserver (Apache, [NGINX](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04-fr))
* `curl`
* `tar`
* `unzip`
* `git`
* `composer`

### Installation de composer

``` bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer --1
```

## Download Files
La première étape de ce processus consiste à créer le dossier dans lequel le panneau sera installé, puis à nous déplacer dans ce dossier nouvellement créé.
dossier nouvellement créé. Vous trouverez ci-dessous un exemple de la manière d'effectuer cette opération.

``` bash
mkdir -p /var/www/evolvinghcs
cd /var/www/evolvinghcs
```

Une fois que vous avez créé un nouveau répertoire pour le Panel et que vous vous y êtes installé, vous devrez télécharger les fichiers du Panel. Pour cela,
C'est aussi simple que d'utiliser `curl` pour télécharger notre contenu pré-packagé. Une fois qu'il est téléchargé, vous devez décompresser l'archive
et ensuite définir les permissions correctes sur les répertoires `storage/` et `bootstrap/cache/`. Ces répertoires
nous permettent de stocker des fichiers et de conserver un cache rapide pour réduire les temps de chargement.

``` bash
curl -Lo evolving.tar.gz https://github.com/nicolasbaud/EvolvingHCS-clientarea/archive/refs/tags/v1.0.0-beta.tar.gz
tar --strip-components=1 -xzvf evolving.tar.gz
chmod -R 755 storage/* bootstrap/cache/
```

## Installation

Maintenant que tous les fichiers ont été téléchargés, nous devons configurer certains aspects fondamentaux du Panel.

```
$ mysql -u root -p
mysql> CREATE USER 'evolvinghcs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'A secure password';
mysql> CREATE DATABASE panel;
mysql> GRANT ALL ON panel.* TO 'evolvinghcs'@'localhost' WITH GRANT OPTION;
```

Puis

``` bash
cp .env.example .env
composer install --no-dev --optimize-autoloader
```

### Environment Configuration
L'environnement de base de EvolvingHCS est facilement configuré à l'aide de quelques commandes CLI différentes intégrées à l'application. Cette étape
couvrira la configuration d'éléments tels que les sessions, la mise en cache, les informations d'identification de la base de données et l'envoi d'e-mails.

``` bash
bash <(curl https://api.evolving-hcs.com/install/setup.sh)
```

### Add The First User
Vous devrez ensuite créer un utilisateur administratif afin de pouvoir vous connecter au panneau. Pour ce faire, exécutez la commande ci-dessous.

``` bash
php artisan make:administrator
```

### Set Permissions
La dernière étape du processus d'installation consiste à définir les autorisations correctes sur les fichiers du Panel afin que le serveur Web puisse les utiliser correctement.

``` bash
# If using NGINX or Apache (not on CentOS):
chown -R www-data:www-data * 
# If using NGINX on CentOS:
chown -R nginx:nginx *
# If using Apache on CentOS
chown -R apache:apache *
```

## Queue Listeners
Nous utilisons des files d'attente pour rendre l'application plus rapide et gérer l'envoi d'e-mails et d'autres actions en arrière-plan. Vous devrez configurer le travailleur de la file d'attente pour que ces actions soient traitées.
### Configuration Crontab
La première chose que nous devons faire est de créer une nouvelle tâche cron qui s'exécute toutes les minutes pour traiter des tâches spécifiques à Pterodactyl, telles que le nettoyage de session et l'envoi de tâches planifiées aux démons. Vous voudrez ouvrir votre crontab en utilisant sudo crontab -e puis coller la ligne ci-dessous.

```bash
* * * * * php /var/www/evolvinghcs/artisan schedule:run >> /dev/null 2>&1
```

#### Ensuite vous devez configurer un vhost pour votre serveur web, pour cela on vous laisse faire :)