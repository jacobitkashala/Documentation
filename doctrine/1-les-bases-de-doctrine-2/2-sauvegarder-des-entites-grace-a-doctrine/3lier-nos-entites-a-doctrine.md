
Une bonne partie du fonctionnement de *Doctrine* est basée sur la configuration des entités. Pour ce faire nous disposons de quatre (4) moyens différents :

- les fichiers [YAML](http://docs.doctrine-project.org/en/latest/reference/yaml-mapping.html) ;
- les fichiers [XML](http://docs.doctrine-project.org/en/latest/reference/xml-mapping.html) ;
- du code [PHP](http://docs.doctrine-project.org/en/latest/reference/php-mapping.html) ;
- et pour finir [les annotations PHP](http://docs.doctrine-project.org/en/latest/reference/annotations-reference.html).

[[information]]
| Les annotations PHP sont des blocs de commentaires qui permettent de rajouter un ensemble de métadonnées à du code. Elles peuvent être présentes sur les attributs des classes, les noms des méthodes et les noms des classes. Et elles sont utilisées par beaucoup de librairies PHP comme PHPUnit (`@dataProvider`, `@expectedException`, etc.) ou encore [le framework Symfony](http://symfony.com/) (`@Route`, `@Security`, etc.).

Nous utiliserons les annotations tout le long du cours et c'est d'ailleurs la technique la plus répandue dans le monde PHP. Mais il faut garder en tête qu'il est relativement simple de passer d'un type de configuration à un autre.

Les annotations ont un grand atout car elles seront directement sur les entités que nous voulons décrire. Il sera donc facile de faire le pont entre nos entités et leurs correspondants dans notre base de données. Ainsi, toutes les modifications du modèle de données seront concentrées à un seul endroit de notre code ce qui facilitera sa prise en main.

# Déclarer une entité

Toutes les annotations de *Doctrine* se trouvent dans l'espace de nom `Doctrine\ORM\Mapping`. Pour une meilleure lisibilité et un code plus concis, nous utiliserons l'alias `ORM`.
Pour déclarer une entité, il suffit d'utiliser l'annotation `Entity`. Avec cette annotation, *Doctrine* sait qu'il doit prendre en compte cette classe.

Notre code devient maintenant :
```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
*/
class User
{
    protected $id;

    protected $firstname;

    protected $lastname;

    protected $role;

    // getters et setters
}
```

[[attention]]
| Toutes les entités doivent avoir l'annotation `Entity`. C'est le lien **obligatoire** entre notre code PHP et *Doctrine*.

# Configurer l'entité

Maintenant que l'entité est *déclarée*, nous pouvons maintenant la configurer à notre guise.
Considérons donc que les identifiants des utilisateurs sont des entiers auto-incrémentés et que le nom, prénom et le rôle sont des chaines de caractères.

Toutes ses attributs sont déclarés en utilisant l'annotation `Column`. Cette annotation comporte plusieurs attributs dont `type` qui permet de définir le type de l'attribut.

*Doctrine* supporte beaucoup de [type de données](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#reference-mapping-types) et on peut citer entre autres :

- **string** pour faire correspondre un *VARCHAR* en SQL en chaîne de caractères PHP ;
- **text** pour faire correspondre un *LONGTEXT* en SQL en chaîne de caractères PHP ;
- **integer** pour faire correspondre un *INT* en SQL en entier PHP ;
- **boolean** pour faire correspondre un *TINYINT* en SQL en booléen PHP ;
- **datetime** pour faire correspondre un *DATETIME* en SQL en date PHP.

La configuration de l'entité devient :

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
*/
class User
{
    /**
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="string")
    */
    protected $firstname;

    /**
    * @ORM\Column(type="string")
    */
    protected $lastname;

    /**
    * @ORM\Column(type="string")
    */
    protected $role;

    // ...
}
```

Maintenant, nous devons déclarer l'identifiant comme étant la clé primaire auto-incrémentée.
Il existe une annotation Doctrine nommée `Id` qui permet de déclarer un attribut comme étant la clé primaire et l'annotation `GeneratedValue` permet quant à elle de gérer l'auto-incrémentation de celui-ci.

La configuration finale devient donc :

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
*/
class User
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="string")
    */
    protected $firstname;

    /**
    * @ORM\Column(type="string")
    */
    protected $lastname;

    /**
    * @ORM\Column(type="string")
    */
    protected $role;

    // ...
}
```

Nous avons, à présent, une entité configurée comme il faut mais pour l'instant notre base de données n'a pas changé. Et c'est là que les commandes *Doctrine* entrent en jeu. Nous pouvons commencer par valider nos annotations en lançant la commande :

```bash
vendor/bin/doctrine orm:validate-schema
[Mapping]  OK - The mapping files are correct.
[Database] FAIL - The database schema is not in sync with the current mapping file.
```

*Doctrine* nous dit que les annotations sont correctes mais la base de données n'est pas synchronisé avec notre configuration.

Une autre commande nous permet de mettre à jour notre base de données ou de récupérer les requêtes SQL que *Doctrine* va exécuter pour faire la mise à jour.

Voyons la requête SQL que *Doctrine* va exécuter avec cette configuration :

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql

CREATE TABLE User (id INT AUTO_INCREMENT NOT NULL, firstname VARCHAR(255) NOT NULL, lastname VARCHAR(255)
NOT NULL, role VARCHAR(255) NOT NULL, PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci
ENGINE = InnoDB;
```

Pour l'instant, nous ne faisons qu'afficher la requête qui sera exécutée. Cela permet de faire une derniére vérification de l'ensemble des modifications que nous voulons appliquées.

Le nom de la table que *Doctrine* veut créer est `User`. Ce nom est exactement le même que celui de l'entité.

Mais par convention, les noms des tables SQL sont souvent en minuscules et au pluriel. Heureusement, avec l'annotation `Table`, nous pouvons définir le nom de chaque table. La configuration de l'entité devient maintenant :

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="users")
*/
class User
{
    // ...
}
```

La configuration étant satisfaisante, nous pouvons créer la table des utilisateurs. Notez la présence de l'option `--force` qui permet d'exécuter la requête SQL.

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force

CREATE TABLE users (id INT AUTO_INCREMENT NOT NULL, firstname VARCHAR(255) NOT NULL, lastname VARCHAR(255)
NOT NULL, role VARCHAR(255) NOT NULL, PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci 
ENGINE = InnoDB;

-- Updating database schema...
-- Database schema updated successfully! "1" query was executed
```

Nous revalidons l'état de notre configuration.

```bash
 vendor/bin/doctrine orm:validate-schema
[Mapping]  OK - The mapping files are correct.
[Database] OK - The database schema is in sync with the mapping files.
```

![Création de la table users](https://zestedesavoir.com/media/galleries/3902/828123f5-33d9-4f3e-9c3b-c21424a5fad2.png)

Notre base de données est maintenant synchronisée !


Par défaut, les noms des colonnes sont créés en se basant sur les noms des attributs des entités. Mais nous pouvons les décolérer en utilisant l'attribut `name` dans les annotations `Column`.

Ainsi `@ORM\Column(type="string", name="my_column")` créerait une colonne nommée `my_column` peut importe le nom de l'attribut sur lequel il est situé.

[[information]]
| Il est possible de supprimer la contrainte `NOT NULL` sur un attribut de notre entité en spécifiant le paramètre `nullable` de l'annotation `Column` à `true`. Ainsi, cet attribut pourra être nul (valeur à `null`).

Bref, nous avons là une configuration fonctionnelle mais qui repose sur un grand nombre de paramètre par défaut de *Doctrine*. Mais il est toujours possible de personnaliser à souhait notre configuration.