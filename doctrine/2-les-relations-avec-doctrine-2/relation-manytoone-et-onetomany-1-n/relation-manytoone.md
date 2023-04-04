Nous devons avoir en tête le schéma que nous désirons avant de pouvoir le réaliser avec *Doctrine*.

Notre modèle de données pour intégrer les questions et leurs réponses ressemblerait à :

![Relation ManyToOne :  Plusieurs réponses associées à une question](https://zestedesavoir.com/media/galleries/3902/07881a7d-eddb-45d5-91ab-fb10a6d3c988.png)

Un tel résultat peut être obtenu avec *Doctrine* en utilisant l'annotation `ManyToOne`. Prenons une réponse ayant comme attribut un libellé et la question à laquelle elle est associée. Une question aura, quant à elle, un libellé et une liste de réponses possibles.

[[question]]
| Mais où placer l'annotation `ManyToOne`, dans l'entité *question* ou dans l'entité *réponse* ?

Cette annotation est souvent source de problèmes et d'incompréhensions. Il existe d'ailleurs plusieurs astuces pour l'utiliser à bon escient.

Reprenons notre exemple :

- Une réponse est liée à **une seule (one)** question ;
- Une question peut avoir **plusieurs (many)** réponses.

![Relation entre Question et Réponse](https://zestedesavoir.com/media/galleries/3902/f491b4ff-21fc-4543-abb8-7c361a663f80.png)

Dans une relation `ManyToOne`, le **many** qualifie l'entité qui doit contenir l'annotation. Ici le **many** qualifie les réponses. Donc, pour notre cas, l'annotation doit être dans l'entité *réponse*.

Nos deux entités doivent donc être configurées comme suit :

- la question :

```php
<?php
# src/Entity/Question.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="questions")
*/
class Question
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\Column(type="text")
    */
    protected $wording;

    public function __toString()
    {
        $format = "Question (id: %s, wording: %s)\n";
        return sprintf($format, $this->id, $this->wording);
    }

    // getters et setters
}
```

 - et la réponse :

```php
<?php
# src/Entity/Answer.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="answers")
*/
class Answer
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
    protected $wording;

    /**
    * @ORM\ManyToOne(targetEntity=Question::class)
    */
    protected $question;

    public function __toString()
    {
        $format = "Answer (id: %s, wording: %s)\n";
        return sprintf($format, $this->id, $this->wording);
    }

    // getters et setters
}
```

Il suffit de mettre à jour la base de données à l'aide de l'invite de commande de *Doctrine*.

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE TABLE answers (id INT AUTO_INCREMENT NOT NULL, question_id INT DEFAULT NULL, wording VARCHAR(255)
 NOT NULL, INDEX IDX_50D0C6061E27F6BF (question_id), PRIMARY KEY(id))
 DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE = InnoDB;
CREATE TABLE questions (id INT AUTO_INCREMENT NOT NULL, wording LONGTEXT NOT NULL, PRIMARY KEY(id))
 DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE answers ADD CONSTRAINT FK_50D0C6061E27F6BF FOREIGN KEY (question_id) REFERENCES questions (id);
```
