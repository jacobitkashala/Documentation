
Reprenons notre exemple et considérons qu'un utilisateur peut avoir une seule adresse et que cette adresse ne peut être liée qu'à un seul utilisateur.

Dans la base de données, le schéma ressemblerait à :

![Relation OneToOne : Un utilisateur avec une adresse](https://zestedesavoir.com/media/galleries/3902/78d5932b-11eb-44ef-a6e8-6fcfb430bea2.png)

Avec *Doctrine*, nous pouvons obtenir ce résultat avec l'annotation `OneToOne`.
Supposons que notre adresse est comme suit (vous pouvez aussi vous entrainer en créant vous-même une entité *adresse*) :

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
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="string")
    */
    protected $street;

    /**
    * @ORM\Column(type="string")
    */
    protected $city;

    /**
    * @ORM\Column(type="string")
    */
    protected $country;

    public function __toString()
    {
        $format = "Address (id: %s, street: %s, city: %s, country: %s)";
        return sprintf($format, $this->id, $this->street, $this->city, $this->country);
    }
    
    // ... tous les getters et setters
}
```


Nous avons jusque-là rien de nouveau. Pour relier cette adresse à un utilisateur, nous devons modifier l'entité *utilisateur*. Pour des soucis de clarté tous les autres attributs seront masqués.

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
    * @ORM\OneToOne(targetEntity=Address::class)
    */
    protected $address;

    // Ajout de l'adresse à la méthode __toString
    public function __toString()
    {
        $format = "User (id: %s, firstname: %s, lastname: %s, role: %s, address: %s)\n";
        return sprintf($format, $this->id, $this->firstname, $this->lastname, $this->role, $this->address);
    }

}
```

Avec l'annotation `OneToOne`, nous disons à *Doctrine* que notre utilisateur peut être lié à une adresse (grâce à l'attribut `targetEntity`).

[[information]]
| Dans `targetEntity`, il faut spécifier un espace de nom complet. Avec PHP 7, l'utilisation de la constante `class` nous facilite la tâche.

En lançant une mise à jour de la base de données, *Doctrine* génère la nouvelle table pour les adresses et crée la clé étrangère dans la table des utilisateurs.

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE TABLE addresses (id INT AUTO_INCREMENT NOT NULL, street VARCHAR(255) NOT NULL, city VARCHAR(255) NOT NULL, 
country VARCHAR(255) NOT NULL, PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE users ADD address_id INT DEFAULT NULL;
ALTER TABLE users ADD CONSTRAINT FK_1483A5E9F5B7AF75 FOREIGN KEY (address_id) REFERENCES addresses (id);
CREATE UNIQUE INDEX UNIQ_1483A5E9F5B7AF75 ON users (address_id);
```
