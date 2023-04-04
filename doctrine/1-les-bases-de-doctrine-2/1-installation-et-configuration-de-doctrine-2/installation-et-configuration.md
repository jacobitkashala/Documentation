Pour commencer à utiliser *Doctrine*, nous devons mettre en place un mininum de configuration.

# Installation

## La base

La méthode d'installation la plus simple est d'utiliser le gestionnaire de dépendances [Composer](https://getcomposer.org/). Vous pouvez l'installer depuis le [site officiel](https://getcomposer.org/download/).

Toutes les dépendances seront installées dans un dossier vide nommé ***doctrine2-tuto***.
Dans ce dossier, lancer la commande :

```bash
composer require doctrine/orm:^2.5
```

Une fois la dépendance installée, il nous faut quelques lignes de configuration. Le code fourni durant ce cours sera compatible avec PHP 7. Il faudra donc avoir une version de PHP 7.X pour s'assurer du bon fonctionnement des extraits de code.

```php
<?php
# bootstrap.php

require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'vendor', 'autoload.php']);

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$entitiesPath = [
    join(DIRECTORY_SEPARATOR, [__DIR__, "src", "Entity"])
];

$isDevMode = true;
$proxyDir = null;
$cache = null;
$useSimpleAnnotationReader = false;

// Connexion à la base de données
$dbParams = [
    'driver'   => 'pdo_mysql',
    'host'     => 'localhost',
    'charset'  => 'utf8',
    'user'     => 'root',
    'password' => '',
    'dbname'   => 'poll',
];

$config = Setup::createAnnotationMetadataConfiguration(
    $entitiesPath,
    $isDevMode,
    $proxyDir,
    $cache,
    $useSimpleAnnotationReader
);
$entityManager = EntityManager::create($dbParams, $config);

return $entityManager;
```

Cette première étape obligatoire permet d'avoir les bases communes pour utiliser *Doctrine*.

[[information]]
| La variable `$dbParams` désigne les paramètres de connexion que nous allons utiliser. Pour notre cas, nous aurons donc une base de données MySQL en local nommée `poll` avec comme utilisateur `root` sans mot de passe. À ce stade, seul le paramètre `dbParams`, nous intéresse. Nous aurons l'occasion d'aborder et d'expliciter l'utilité des autres paramètres par la suite.

Assurez-vous d'avoir un moteur de base de données installé et configuré correctement avant de passer à l'étape suivante. Si vous voulez tester ce cours rapidement sans beaucoup d'efforts sur ce point, vous pouvez utiliser le pilote SQLite.

Voici quelques paramètres de configuration utilisables selon votre moteur de base de données.

- Pilote MySQL :
```php
<?php
[
    'driver'   => 'pdo_mysql',
    'host'     => 'localhost',
    'charset'  => 'utf8',
    'user'     => 'root',
    'password' => '',
    'dbname'   => 'poll',
];
```

- Pilote SQLite :
```php
<?php
[
    'driver'   => 'pdo_sqlite',
    'path'   => 'data/poll.sqlite'
];
```
Pensez à créer le dossier ***data*** si vous optez pour *SQLite*.

- Pilote PostgreSQL :
```php
<?php
[
    'driver'   => 'pdo_pgsql',
    'host'     => 'localhost',
    'charset'  => 'utf8',
    'user'     => 'root',
    'password' => '',
    'dbname'   => 'poll',
];
```

N'hésitez pas à consulter la documentation officielle du composant [Doctrine DBAL](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html) pour plus de détails sur les paramètres de connexion. 

Le composant *Doctrine DBAL[^dbal]* est une couche par-dessus [PDO[^pdo]](http://php.net/manual/fr/intro.pdo.php) utilisée en interne par *Doctrine ORM* pour accéder à nos bases de données. Nous n'aurons donc pas à l'utiliser directement dans ce cours.

[^dbal]:  Database Abstraction & Access Layer
[^pdo]: PHP Data Objects

## La ligne de commande

La bibliothéque *doctrine/orm* fournit un ensemble de commandes qui nous seront d'une grande aide pendant nos développements. Cependant quelques lignes de configuration sont nécessaires pour les activer. Nous allons donc créer un fichier ***cli-config.php*** pour finaliser la configuration.

```php
<?php
# cli-config.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Doctrine\ORM\Tools\Console\ConsoleRunner;

return ConsoleRunner::createHelperSet($entityManager);
```


[[attention]]
| Le fichier doit obligatoirement s'appeler `cli-config.php` ou alors `config/cli-config.php` pour que les commandes de Doctrine puissent fonctionner.

Il est maintenant possible de vérifier l'installation en lançant la commande :
```bash
vendor/bin/doctrine

Doctrine Command Line Interface 2.5.5-DEV
Usage:
  command [options] [arguments]

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  help                            Displays help for a command
  list                            Lists commands
 dbal
  dbal:import                     Import SQL file(s) directly to Database.
  dbal:run-sql                    Executes arbitrary SQL directly from the command line.
 orm
  orm:clear-cache:metadata        Clear all metadata cache of the various cache drivers.
  orm:clear-cache:query           Clear all query cache of the various cache drivers.
  orm:clear-cache:result          Clear all result cache of the various cache drivers.
```

Un ensemble de commandes sont affichées dans la console. Nous aurons l'occasion de voir en détails l'utilité de chacune d'elles plus tard.

# Configuration

## Configuration des entités

Durant tout le long de cours, nous parlerons souvent d'entités.

[[question]]
| Mais qu'est qu'une entité ?

Une entité désigne tout simplement une classe permettant de faire le lien entre notre application et notre base de données.
Elle représente donc une classe avec un ensemble d'attributs qui seront liés à des colonnes d'une table de notre base de données grâce à *Doctrine*. 

Pour le bon fonctionnement de *Doctrine*, nous devons spécifier le ou les dossiers contenant ses entités. Pour notre cas, nous placerons les entités dans le dossier ***src/Entity***.

```php
<?php
# bootstrap.php

require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'vendor', 'autoload.php']);

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$entitiesPath = [
    join(DIRECTORY_SEPARATOR, [__DIR__, "src", "Entity"])
];

// ...
```

## Configuration de Composer

Pour finir, nous allons aussi configurer *Composer* pour gérer le système d'*autoloading* de notre application. Cette étape n'est certes pas obligatoire pour utiliser *Doctrine* mais pour nous faciliter le travail, nous allons créer une règle d'*autoloading* pour charger via *Composer* toutes nos classes.

Modifions le fichier `composer.json` pour charger le préfixe `Tuto` depuis le dossier ***src***.

```json
{
    "require": {
        "doctrine/orm": "^2.5"
    },
    "autoload": {
        "psr-4": {
            "Tuto\\": "src/"
        }
    }
}
```

Nous pouvons maintenant recharger la configuration de l'*autoloader* de *Composer* avec la commande :

```bash
composer dump-autoload
```

L'arborescence de notre projet est maintenant :

```bash
├── bootstrap.php
├── cli-config.php
├── composer.json
├── composer.lock
├── src
└── vendor
```
