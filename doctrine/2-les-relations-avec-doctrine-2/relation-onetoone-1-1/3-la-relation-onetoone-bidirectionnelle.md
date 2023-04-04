
Avec notre modèle actuel, lorsque nous avons une entité *utilisateur*, nous pouvons trouver l'adresse qui lui est associée. Par contre, lorsque nous avons une entité *adresse*, nous ne sommes pas en mesure de récupérer l'utilisateur associé. La relation que nous avons configurée est dite **unidirectionnelle** car seul un des membres de la relation peut faire référence à l'autre.

Dans certains cas, il est possible d'avoir une relation dite bidirectionnelle où chacun des deux membres peut faire référence à l'autre.

Pour ce faire, nous devons mettre à jour les annotations des entités.

Nous allons d'abord modifier l'adresse pour rajouter l'information relative à l'utilisateur.

```php
<?php
# src/Entity/Address.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="addresses")
*/
class Address
{
    // ...

    /**
    * @ORM\OneToOne(targetEntity=User::class, mappedBy="address")
    */
    protected $user;

    // ...
}
```

L'attribut `mappedBy` est obligatoire et elle permet de dire à *Doctrine* que cette relation est bidirectionnelle et que l'attribut utilisé dans l'autre  côté de la relation est `address`.


Dans la même logique, au niveau de l'utilisateur, nous devons mettre en place un attribut `inverseBy` pour signifier à *Doctrine* que nous utilisons une relation bidirectionnelle. La valeur de ce paramètre désigne le nom de l'attribut dans l'autre côté de la relation.

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
    */
    protected $address;

    // ...
}
```

Ces modifications n'affectent pas la configuration de la base de données mais permettent au niveau applicatif d'accéder plus facilement à certaines informations.

Pour notre exemple actuel, avoir une relation bidirectionnelle n'est pas pertinente car l'adresse ne sera jamais utilisée sans l'utilisateur. Nous pouvons donc nous en passer.