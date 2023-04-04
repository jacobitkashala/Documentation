# Énoncé

Pour pratiquer ce que nous venons de voir, nous allons créer une entité _sondage_ avec deux attributs :

- son titre : une chaîne de caractères qui permet de décrire le sondage ;
- sa date de création : une date PHP permettant de savoir la date de création du sondage.

Le sondage est constitué d'une liste de questions. La relation entre les deux entités doit être bidirectionnelle mais aucune opération de cascade ne doit être définie.

![Relation ManyToOne : Plusieurs questions associées à un sondage](https://zestedesavoir.com/media/galleries/3902/29d3f66c-db60-41e9-8547-86dd0ceb6bc6.png)

# Proposition de solution

Nous allons créer une entité nommé `Poll` pour représenter le sondage et éditer l'entité _question_ pour le lier au sondage.

Si nous réécrivons notre exemple :

- Une question est liée à **un seul (one)** sondage ;
- Un sondage peut avoir **plusieurs (many)** questions.

![Relation entre Question et Sondage](https://zestedesavoir.com/media/galleries/3902/93b8b913-5d28-4b39-a885-54e7f0a523f4.png)

> Dans une relation `ManyToOne`, le **many** qualifie l'entité qui doit contenir l'annotation.

Donc, pour notre cas, l'annotation doit être dans l'entité _question_.

La configuration des entités est donc :

```php
<?php
# src/Entity/Poll.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="polls")
 */
class Poll
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
  protected $title;

  /**
   * @ORM\Column(type="datetime")
   */
  protected $created;

  /**
   * @ORM\OneToMany(targetEntity=Question::class, mappedBy="poll")
   */
  protected $questions;

  public function __construct()
  {
    $this->questions = new ArrayCollection();
  }

  public function __toString()
  {
    $format = "Poll (id: %s, title: %s, created: %s)\n";
    return sprintf(
      $format,
      $this->id,
      $this->title,
      $this->created->format(\Datetime::ISO8601)
    );
  }

  // ...

  public function addQuestion(Question $question)
  {
    // Toujours maintenir la relation cohérente
    $this->questions->add($question);
    $question->setPoll($this);
  }
}
```

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
   * @ORM\ManyToOne(targetEntity=Poll::class, inversedBy="questions")
   */
  protected $poll;

  // ...
}
```

Mettons à jour la base de données :

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE TABLE polls (id INT AUTO_INCREMENT NOT NULL, title VARCHAR(255) NOT NULL, created DATETIME NOT NULL,
 PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE questions ADD poll_id INT DEFAULT NULL;
ALTER TABLE questions ADD CONSTRAINT FK_8ADC54D53C947C0F FOREIGN KEY (poll_id) REFERENCES polls (id);
CREATE INDEX IDX_8ADC54D53C947C0F ON questions (poll_id);
```

Nous pouvons maintenant tester la création d'un sondage avec quelques questions. La seule nouveauté ici sera l'utilisation des dates PHP.

En effet, _Doctrine_ transforme automatiquement les dates pour nous. Le seul élément à prendre en compte est le fuseau horaire. Vu que MySQL ne stocke pas le fuseau horaire dans ses dates, nous devons définir dans PHP un fuseau horaire identique pour toutes les applications qui pourraient d'utiliser les mêmes données.

Nous pouvons utiliser la fonction PHP [date_default_timezone_set](http://php.net/manual/fr/function.date-default-timezone-set.php) ou le paramètre d'initialisation [date.timezone](http://php.net/manual/fr/datetime.configuration.php#ini.date.timezone).

```php
<?php
# create-poll.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [
  __DIR__,
  'bootstrap.php',
]);

use Tuto\Entity\Poll;
use Tuto\Entity\Question;
use Tuto\Entity\Answer;

$poll = new Poll();

$poll->setTitle('Doctrine 2, ça vous dit ?');
$poll->setCreated(new \Datetime('2017-03-03T08:00:00Z'));

# Question 1
$questionOne = new Question();
$questionOne->setWording('Doctrine 2 est-il un bon ORM ?');

$yes = new Answer();
$yes->setWording('Oui, bien sûr !');
$questionOne->addAnswer($yes);

$no = new Answer();
$no->setWording('Non, peut mieux faire.');
$questionOne->addAnswer($no);

# Ajout de la question au sondage
$poll->addQuestion($questionOne);

# Question 2
$questionTwo = new Question();
$questionTwo->setWording("Doctrine 2 est-il facile d'utilisation ?");

$yesDoc = new Answer();
$yesDoc->setWording('Oui, il y a une bonne documentation !');
$questionTwo->addAnswer($yesDoc);

$yesTuto = new Answer();
$yesTuto->setWording('Oui, il y a de bons tutoriels !');
$questionTwo->addAnswer($yesTuto);

$no = new Answer();
$no->setWording('Non.');
$questionTwo->addAnswer($no);

# Ajout de la question au sondage
$poll->addQuestion($questionTwo);

$entityManager->persist($questionOne);
$entityManager->persist($questionTwo);
$entityManager->persist($poll);

$entityManager->flush();

echo $poll;
```

Une fois que le sondage est créé, nous pouvons le récupérer et l'afficher. Grâce aux relations que nous avons définies, _Doctrine_ peut chercher toutes les informations dont nous avons besoin en se basant juste sur le sondage.
Voyez donc par vous-même :

```php
<?php
# get-poll.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [
  __DIR__,
  'bootstrap.php',
]);

use Tuto\Entity\Poll;

$pollRepo = $entityManager->getRepository(Poll::class);

$poll = $pollRepo->find(1);

echo $poll;
foreach ($poll->getQuestions() as $question) {
  echo '- ', $question;

  foreach ($question->getAnswers() as $answer) {
    echo '-- ', $answer;
  }
}
```

```txt
Poll (id: 1, title: Doctrine 2, ça vous dit ?, created: 2017-03-03T08:00:00+0000)
- Question (id: 2, wording: Doctrine 2 est-il un bon ORM ?)
-- Answer (id: 3, wording: Oui, bien sûr !)
-- Answer (id: 4, wording: Non, peut mieux faire.)
- Question (id: 3, wording: Doctrine 2 est-il facile d'utilisation ?)
-- Answer (id: 5, wording: Oui, il y a une bonne documentation !)
-- Answer (id: 6, wording: Oui, il y a de bons tutoriels !)
-- Answer (id: 7, wording: Non.)
```
