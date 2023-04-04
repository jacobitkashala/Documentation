
Dans le cas des questions et réponses, il est légitime de vouloir accéder aux réponses depuis l'entité *question* (pour afficher rapidement les réponses associées à une question par exemple).

Comme pour la relation `OneToOne`, nous pouvons rendre une relation `ManyToOne` bidirectionnelle en utilisant l'annotation *miroir* `OneToMany`. La configuration est assez proche de celle de l'annotation `OneToOne`. Nous pouvons même activer les opérations de cascade pour créer une question et toutes les réponses associées plus facilement.

Ainsi, à l'image de toutes les relations bidirectionnelles, chacune des deux entités doit maintenant être configurée pour faire référence à l'autre.

La configuration finale est donc :

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
    // ...

    /**
    * @ORM\ManyToOne(targetEntity=Question::class, inversedBy="answers")
    */
    protected $question;

   // ...
}

```


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
   // ...

   /**
    * @ORM\OneToMany(targetEntity=Answer::class, cascade={"persist", "remove"}, mappedBy="question")
    */
    protected $answers;

    // ...
}

```

Testons le tout en créant une question et plusieurs réponses associées.

Pour créer une question, le code est simple.

```php
<?php
# create-question.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\Question;
use Tuto\Entity\Answer;

$question = new Question();

$question->setWording("Doctrine 2 est-il un bon ORM ?");

$entityManager->persist($question);
$entityManager->flush();

echo $question;
```

[[question]]
| Mais comment relier une réponse à une question ?

L'attribut `answers` de la classe `Question` représente une liste de réponses. Avec *Doctrine*, nous pouvons représenter une collection grâce à la classe `ArrayCollection`. 
Cette classe permettra à *Doctrine* de gérer convenablement tous les changements qui pourront subvenir sur la collection.

L'API de la classe `ArrayCollection` est pratique pour manipuler des listes (ajout, suppression, recherche d'un élément).

Il est d'ailleurs possible d'utiliser le package [doctrine/collections](https://github.com/doctrine/collections) dans des projets pour profiter des fonctionnalités des collections sans installer l'ORM *Doctrine*.

Nous allons donc modifier l'entité *question* pour rajouter ces modifications.

```php
<?php
# src/Entity/Question.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="questions")
*/
class Question
{
    // ...

   /**
    * @ORM\OneToMany(targetEntity=Answer::class, cascade={"persist", "remove"}, mappedBy="question")
    */
    protected $answers;

    public function __construct()
    {
        $this->answers = new ArrayCollection();
    }
    // ...

    public function getAnswers()
    {
        return $this->answers;
    }
     
    public function addAnswer(Answer $answer)
    {
        $this->answers->add($answer);
        $answer->setQuestion($this);
    }
}
```

Dans la méthode `addAnswer`, nous avons rajouté une petite logique qui permet de maintenir l'application dans un état cohérent. Lorsqu'une réponse est associée à une question, cette question est elle aussi automatiquement liée à la réponse.

Cette liaison est obligatoire pour le bon fonctionnement de la cascade d'opération de *Doctrine* (vous pourrez consulter la partie annexe de ce cours pour une explication en détail - *Owning side - Inverse side*). Pour chaque relation faisant intervenir une collection, il faudra y porter une attention particulière.

Nous pouvons maintenant compléter la création de la question.

```php
<?php
# create-question.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\Question;
use Tuto\Entity\Answer;

$question = new Question();

$question->setWording("Doctrine 2 est-il un bon ORM ?");

$yes = new Answer();
$yes->setWording("Oui, bien sûr !");

$question->addAnswer($yes);

$no = new Answer();
$no->setWording("Non, peut mieux faire.");

$question->addAnswer($no);

$entityManager->persist($question);
$entityManager->flush();

echo $question;
```

Grâce à la cascade des opérations de sauvegarde, nous ne sommes pas obligés de persister les réponses. *Doctrine* regarde l'état de la collection et fait le nécessaire pour que les réponses soient sauvegardées correctement.

![Insertion d'une question](https://zestedesavoir.com/media/galleries/3902/3a7e61d5-f39f-417d-b854-917077867cc3.png)

![Insertion des réponses](https://zestedesavoir.com/media/galleries/3902/02bb1d9a-cb87-4b27-8076-283ccde7c271.png)