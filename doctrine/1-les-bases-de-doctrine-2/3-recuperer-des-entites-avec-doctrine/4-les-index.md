
Nous avons actuellement une base de données peu volumineuse mais la quantité de données dans une application en production peut rapidement croître.

Ainsi les performances des requêtes de lecture sont à prendre en compte dès la conception du modèle de données. Avec *Doctrine*, nous pouvons mettre en place des index afin d'améliorer ceux-ci. Les index SQL sont des structures permettant aux bases de données de référencer une ou plusieurs colonnes et ainsi accélérer les recherches effectuées sur celles-ci.

Pour déclarer des index grâce à *Doctrine*, nous pouvons utiliser l'annotation `Index`. Elle permet de déclarer un index en spécifiant son nom et la liste des colonnes indexées.

[[erreur]]
| Les noms des colonnes **doivent être ceux dans la table SQL** et non pas le nom des attributs de la classe.

Si par exemple, nous voulions rajouter un index sur le nom et prénom des utilisateurs et un autre sur le rôle de ceux-ci, nous aurions comme configuration :

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(
    name="users",
    indexes={
        @ORM\Index(name="search_firstname_lastname", columns={"firstname", "lastname"}),
        @ORM\Index(name="search_role", columns={"role"})
    }
 )
*/
class User
{
   // ...
}
```

Bien sûr pour que les index soient créés, il faut mettre à jour la base de données avec la commande :

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE INDEX search_firstname_lastname ON users (firstname, lastname);
CREATE INDEX search_role ON users (role);
```
