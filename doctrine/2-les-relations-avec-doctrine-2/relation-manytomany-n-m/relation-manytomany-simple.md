
Nous pouvons commencer par essayer de recenser tous les utilisateurs qui ont participé à un sondage.

Pour obtenir un schéma de données permettant de gérer cette contrainte, nous sommes obligés d'avoir une table de jointure.

![Relation ManyToMany : Plusieurs utilisateurs , plusieurs sondages](https://zestedesavoir.com/media/galleries/3902/d8ccaa75-8f68-48e1-9fda-026edd083543.png)


La configuration de l'annotation `ManyToMany` est très simple. La seule question que nous devons nous poser est : 

[[question]]
| Depuis quelle entité devrons-nous faire référence à l'autre ?

Dans notre cas, nous allons faire un choix **complétement arbitraire**. Depuis un utilisateur, nous serons en mesure de voir tous les sondages auxquels il a participé. L'annotation `ManyToMany` sera donc configurée sur l'entité *utilisateur*. 

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

// ...
class User
{
    //...

    /**
    * @ORM\ManyToMany(targetEntity=Poll::class)
    */
    protected $polls;

    public function __construct()
    {
        $this->polls = new ArrayCollection();
    }

    // ...
}
```

Il faut noter que si nous voulons, par la suite, accéder aux utilisateurs depuis un sondage, nous pourrons rendre la relation bidirectionnelle.

Générons le code SQL avec la commande *Doctrine* :

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql

CREATE TABLE user_poll (user_id INT NOT NULL, poll_id INT NOT NULL, INDEX IDX_FE3DB68CA76ED395 (user_id), 
INDEX IDX_FE3DB68C3C947C0F (poll_id), PRIMARY KEY(user_id, poll_id)) DEFAULT CHARACTER SET utf8 
COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE user_poll ADD CONSTRAINT FK_FE3DB68CA76ED395 FOREIGN KEY (user_id) 
REFERENCES users (id) ON DELETE CASCADE;
ALTER TABLE user_poll ADD CONSTRAINT FK_FE3DB68C3C947C0F FOREIGN KEY (poll_id) 
REFERENCES polls (id) ON DELETE CASCADE;
```

Le nom de la table de jointure est `user_poll`. Il n'est pas assez explicite et ne permet pas de voir de manière claire la nature de la relation. En plus, nous avons défini une convention au début de ce cours pour mettre tous les noms de table au pluriel.

Mais heureusement, nous pouvons le personnaliser en utilisant l'annotation `JoinTable`. 

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
    * @ORM\ManyToMany(targetEntity=Poll::class)
    * @ORM\JoinTable(name="participations")
    */
    protected $polls;

    public function __construct()
    {
        $this->polls = new ArrayCollection();
    }

    // ...
}
```

Le nom de la table est maintenant correct. Mais nous n'allons pas encore la créer.