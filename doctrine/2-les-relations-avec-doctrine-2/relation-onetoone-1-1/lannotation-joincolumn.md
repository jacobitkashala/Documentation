
Si nous consultons la base de données, nous pouvons voir que la colonne qui porte la clé étrangère s'appelle `address_id`. *Doctrine* a choisi automatique ce nom en se basant sur l'attribut `address` de notre entité *utilisateur*.

De plus, avec la configuration actuelle, il est possible d'avoir un utilisateur sans adresse (colonne à `NULL`). Nous pouvons personnaliser ces informations en utilisant l'annotation `JoinColumn`.

Cette annotation nous permet entre autres :

- de modifier le nom de la colonne portant la clé étrangère avec son attribut `name` ;
- ou de rajouter une contrainte de nullité avec l'attribut `nullable`.

Avec comme configuration :

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

    /**
    * @ORM\OneToOne(targetEntity=Address::class, cascade={"persist", "remove"}, inversedBy="user")
    * @ORM\JoinColumn(name="address", nullable=false)
    */
    protected $address;

    // ...
}
```

... nous aurions eu comme requête SQL :

```sql
ALTER TABLE users ADD address INT NOT NULL;
```

au lieu de :

```sql
ALTER TABLE users ADD address_id INT DEFAULT NULL;
```

*Doctrine* utilise beaucoup de valeurs par défaut mais nous laisse le choix de les personnaliser à travers les différentes annotations à notre disposition.
