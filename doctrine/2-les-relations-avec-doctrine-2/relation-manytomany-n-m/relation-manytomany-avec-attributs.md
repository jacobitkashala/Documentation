
[[question]]
| Comment rajouter la date à laquelle l'utilisateur a participé au sondage ?

Dans un modèle purement SQL, cette information doit se retrouver dans la table ***participations***. Mais avec l'ORM, nous avons juste les entités utilisateur et sondage. La date ne peut être lié à aucunes des deux.

En effet, si la date de participation est liée à l'utilisateur, nous ne pourrons conserver qu'une seule date de participation par utilisateur (celui du dernier sondage).

Et si cette date est liée au sondage, nous ne pourrons conserver que la date de participation d'un seul utilisateur (celui du dernier utilisateur ayant participé au sondage). **Nous atteignons là une limite de la relation *ManyToMany*.** 

Mais le problème peut être résolu en utilisant une double relation `ManyToOne`. Considérons que le fait de participer à un sondage est matérialisé par une entité *participation*.

Ainsi, un utilisateur peut avoir **plusieurs** participations (en répondant à plusieurs sondages). Et une participation est créée par **un seul** utilisateur.

![Relation entre Participation et Utilisateur](https://zestedesavoir.com/media/galleries/3902/49b2fac4-8d83-4529-93e6-2214c531a3cf.png)

Donc entre une participation et un utilisateur, nous avons une relation `ManyToOne`. Mais cela ne s'arrête pas là !

Un sondage peut avoir **plusieurs** participations (plusieurs utilisateurs peuvent répondre à un même sondage). Une participation est liée à **un** sondage. 

![Relation entre Participation et Sondage](https://zestedesavoir.com/media/galleries/3902/f2a94284-b4e9-43d8-bae8-3f86ed7552a4.png)

Nous avons donc aussi une relation `ManyToOne` entre une participation et un sondage.

Nous pouvons maintenant réécrire la relation `ManyToMany`. Il faudra d'abord créer une entité *participation* qui sera liée aux entités *utilisateur* et *sondage*. 

```php
<?php
# src/Entity/Participation.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="participations",
    uniqueConstraints={
        @ORM\UniqueConstraint(name="user_poll_unique", columns={"user_id", "poll_id"})
    }
  )
*/
class Participation
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="datetime")
    */
    protected $date;

    /**
    * @ORM\ManyToOne(targetEntity=User::class, inversedBy="participations")
    */
    protected $user;

    /**
    * @ORM\ManyToOne(targetEntity=Poll::class)
    */
    protected $poll;

    public function __toString()
    {
        $format = "Participation (Id: %s, %s, %s)\n";
        return sprintf($format, $this->id, $this->user, $this->poll);
    }

   // ...
}
```

Il y a une petite subtilité dans la configuration de celle-ci. Nous avons rajouté une contrainte d'unicité sur les champs `user` et `poll` en utilisant l'annotation `UniqueConstraint`. Son utilisation est proche de celle de l'annotation `Index`.

Avec cette contrainte, nous interdisons à un utilisateur de répondre plusieurs fois à un même sondage.

Pour l'entité *utilisateur*, nous allons juste reconfigurer l'attribut `polls`.

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

// ...
class User
{
    // ...

    /**
    * @ORM\OneToMany(targetEntity=Participation::class, mappedBy="user")
    */
    protected $participations;

    public function __construct()
    {
        $this->participations = new ArrayCollection();
    }
     
    // ...
}
```


[[attention]]
| Pour l'entité *sondage*, puisque la relation est unidirectionnelle, nous avons aucune modification à faire dessus.

En relançant la commande de mise à jour de la base de données, vous pouvez voir que le code SQL généré est exactement le même que celui dans la relation `ManyToMany`.

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql 
CREATE TABLE participations (id INT AUTO_INCREMENT NOT NULL, user_id INT DEFAULT NULL, poll_id INT DEFAULT NULL, 
INDEX IDX_FDC6C6E8A76ED395 (user_id), INDEX IDX_FDC6C6E83C947C0F (poll_id),
 UNIQUE INDEX user_poll_unique (user_id, poll_id),
 PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE participations ADD CONSTRAINT FK_FDC6C6E8A76ED395 FOREIGN KEY (user_id) REFERENCES users (id);
ALTER TABLE participations ADD CONSTRAINT FK_FDC6C6E83C947C0F FOREIGN KEY (poll_id) REFERENCES polls (id);
```


[[information]]
| Toute relation `ManyToMany` peut ainsi être décomposée en deux relations `ManyToOne`.


![Décomposition de la relation ManyToMany](https://zestedesavoir.com/media/galleries/3902/4037c86a-f70d-4b4d-90f1-306c378754cf.png)


Revenons maintenant à notre problème initial : Comment rajouter la date à laquelle l'utilisateur a participé à un sondage ?

Avec notre nouveau modèle de données, la réponse est simple. Il suffit de rajouter la date de participation à l'entité *participation*.

```php
<?php
# src/Entity/Participation.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="participations",
    uniqueConstraints={
        @ORM\UniqueConstraint(name="user_poll_unique", columns={"user_id", "poll_id"})
    }
  )
*/
class Participation
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="datetime")
    */
    protected $date;

    /**
    * @ORM\ManyToOne(targetEntity=User::class, inversedBy="participations")
    */
    protected $user;

    /**
    * @ORM\ManyToOne(targetEntity=Poll::class)
    */
    protected $poll;

    public function __toString()
    {
        $format = "Participation (Id: %s, %s, %s)\n";
        return sprintf($format, $this->id, $this->user, $this->poll);
    }

    // ...
}
```

Nous pouvons maintenant mettre à jour la base de données.

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE TABLE participations (id INT AUTO_INCREMENT NOT NULL, user_id INT DEFAULT NULL, poll_id INT DEFAULT NULL,
 date DATETIME NOT NULL, INDEX IDX_FDC6C6E8A76ED395 (user_id), INDEX IDX_FDC6C6E83C947C0F (poll_id),
 UNIQUE INDEX user_poll_unique (user_id, poll_id), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 
COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE participations ADD CONSTRAINT FK_FDC6C6E8A76ED395 FOREIGN KEY (user_id) REFERENCES users (id);
ALTER TABLE participations ADD CONSTRAINT FK_FDC6C6E83C947C0F FOREIGN KEY (poll_id) REFERENCES polls (id);
```