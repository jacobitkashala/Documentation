# Définition et implémentation du schéma

Nous allons comment par modéliser l'élément le plus élémentaire dans la gestion des choix des utilisateurs : un choix pour une question donnée.

Un choix représente le ou les réponses que l'utilisateur a choisies pour une question donnée. De cette description, nous pouvons déduire deux informations :

- un choix est en relation avec **une seule** question du sondage (la question à laquelle l'utilisateur répond) ;
- et elle est en relation avec **une ou plusieurs** réponses (les réponses sont liées à la question à laquelle l'utilisateur répond).

Pour décrire les relations avec *Doctrine*, nous pouvons dire que :

- Un choix est lié à **une** question.
- Une question est liée à **plusieurs** choix (plusieurs utilisateurs peuvent répondre à une même question).

Entre l'entité *choix* et *question*, nous avons une relation `ManyToOne`.

```php
<?php
#src/Entity/Choice.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="choices")
*/
class Choice
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\ManyToOne(targetEntity=Question::class)
    */
    protected $question;

    public function __toString()
    {
        $format = "Choice (id: %s)\n";
        return sprintf($format, $this->id);
    }

    // ...
}
```

Dans la même logique,

- Un choix peut contenir **plusieurs** réponses (pour les questions à choix multiple).
- Une réponse est liée à **plusieurs** choix (plusieurs utilisateurs peuvent choisir les mêmes réponses pour une question donnée).

Nous avons donc une relation `ManyToMany` entre les entités *choix* et *réponse* .

```php
<?php
#src/Entity/Choice.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="choices")
*/
class Choice
{
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    protected $id;

    /**
    * @ORM\ManyToOne(targetEntity=Question::class)
    */
    protected $question;

    /**
    * @ORM\ManyToMany(targetEntity=Answer::class)
    * @ORM\JoinTable(name="selected_answers")
    */
    protected $answers;

    public function __construct()
    {
        $this->answers = new ArrayCollection();
    }

    public function __toString()
    {
        $format = "Choice (id: %s)\n";
        return sprintf($format, $this->id);
    }

    public function addAnswer(Answer $answer)
    {
        if ($answer->getQuestion() == $this->question) {
            $this->answers->add($answer);
        }
    }

    public function getId()
    {
        return $this->id;
    }
     
    public function setId($id)
    {
        $this->id = $id;
    }
     
    public function getQuestion()
    {
        return $this->question;
    }
     
    public function setQuestion($question)
    {
        $this->question = $question;
    }
     
    public function getAnswers()
    {
        return $this->answers;
    }

    public function getParticipation()
    {
        return $this->participation;
    }
     
    public function setParticipation($participation)
    {
        $this->participation = $participation;
    }
}
```

L'annotation `JoinTable` sur l'attribut `answers` nous permet de nommer la table qui contiendra les réponses choisies : `selected_answers`.

Maintenant que le choix est modélisé, nous devons le relier au sondage. En répondant à un sondage, l'utilisateur fait un choix pour chaque question. Pour rappel, nous avons déjà une entité *participation* qui nous permet déjà de relier un utilisateur et le sondage auquel il participe.

La participation a un sondage peut donc être complétée en enregistrant tous les choix de l'utilisateur.

- Une participation contient **plusieurs** choix.
- Un choix est lié à **une** participation.

La relation entre l'entité *participation* et *choix* est une relation *ManyToOne*. Nous allons la rendre bidirectionnelle pour pouvoir récupérer tous les choix depuis la participation d'un utilisateur.

```php
<?php
# src/Entity/Participation.php

namespace Tuto\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

// ...
class Participation
{
    // ...

    /**
    * @ORM\OneToMany(targetEntity=Choice::class, mappedBy="participation")
    */
    protected $choices;

    public function __construct()
    {
        $this->choices = new ArrayCollection();
    }

    // ...

    public function addChoice(Choice $choice)
    {
        $this->choices->add($choice);
        $choice->setParticipation($this);
    }
}
```

La mise à jour de la base de données donne :

```sql
-- vendor/bin/doctrine orm:schema-tool:update --dump-sql --force
CREATE TABLE choices (id INT AUTO_INCREMENT NOT NULL, question_id INT DEFAULT NULL, 
participation_id INT DEFAULT NULL, INDEX IDX_5CE96391E27F6BF (question_id), 
INDEX IDX_5CE96396ACE3B73 (participation_id), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 
COLLATE utf8_unicode_ci ENGINE = InnoDB;
CREATE TABLE selected_answers (choice_id INT NOT NULL, answer_id INT NOT NULL, INDEX IDX_315F9F94998666D1 (choice_id), 
INDEX IDX_315F9F94AA334807 (answer_id), PRIMARY KEY(choice_id, answer_id)) DEFAULT CHARACTER SET utf8 COLLATE 
utf8_unicode_ci ENGINE = InnoDB;
ALTER TABLE choices ADD CONSTRAINT FK_5CE96391E27F6BF FOREIGN KEY (question_id) REFERENCES questions (id);
ALTER TABLE choices ADD CONSTRAINT FK_5CE96396ACE3B73 FOREIGN KEY (participation_id) REFERENCES participations (id);
ALTER TABLE selected_answers ADD CONSTRAINT FK_315F9F94998666D1 FOREIGN KEY (choice_id) REFERENCES choices (id) 
ON DELETE CASCADE;
ALTER TABLE selected_answers ADD CONSTRAINT FK_315F9F94AA334807 FOREIGN KEY (answer_id) REFERENCES answers (id) 
ON DELETE CASCADE;
```

# Tests

Nous allons tester notre implémentation répondant au sondage que nous avons créé dans le chapitre précédent. Pour rappel, voici le contenu du sondage :

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

Pour la première question, l'utilisateur va répondre « Oui, bien sûr ! » et pour la seconde, il choisira les deux réponses « Oui ».

```php
<?php
#create-full-participation.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\Participation;
use Tuto\Entity\Poll;
use Tuto\Entity\User;
use Tuto\Entity\Choice;

$participation = new Participation();

$participation->setDate(new \Datetime("2017-03-04T10:00:00Z"));
$entityManager->persist($participation);
$entityManager->flush(); // Nous devons flusher une premiére fois pour avoir un identifiant pour la participation

$pollRepo = $entityManager->getRepository(Poll::class);
$poll = $pollRepo->find(1);

$userRepo = $entityManager->getRepository(User::class);
$user = $userRepo->find(3);


$participation->setUser($user);
$participation->setPoll($poll);

/*
Poll (id: 1, title: Doctrine 2, ça vous dit ?, created: 2017-03-03T08:00:00+0000)
- Question (id: 2, wording: Doctrine 2 est-il un bon ORM ?)
-- Answer (id: 3, wording: Oui, bien sûr !)
-- Answer (id: 4, wording: Non, peut mieux faire.)
- Question (id: 3, wording: Doctrine 2 est-il facile d'utilisation ?)
-- Answer (id: 5, wording: Oui, il y a une bonne documentation !)
-- Answer (id: 6, wording: Oui, il y a de bons tutoriels !)
-- Answer (id: 7, wording: Non.)
*/

$questions = $poll->getQuestions();

// Choix pour la première question
$questionOne = $questions->get(0);
$answers = $questionOne->getAnswers();

$choice = new Choice();
$choice->setQuestion($questionOne);
$choice->addAnswer($answers->get(0));

$entityManager->persist($choice);
$participation->addChoice($choice);

// Choix pour la deuxième question
$questionTwo = $questions->get(1);
$answers = $questionTwo->getAnswers();

$choice = new Choice();
$choice->setQuestion($questionTwo);
$choice->addAnswer($answers->get(0));
$choice->addAnswer($answers->get(1));

$entityManager->persist($choice);
$participation->addChoice($choice);

$entityManager->flush();

echo $participation;
```

Une fois la participation créée, nous pouvons maintenant afficher toutes les participations de l'utilisateur (avec un peu de formatage pour une meilleure lisibilité).

```php
<?php
# get-user-participations.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$userRepo = $entityManager->getRepository(User::class);
$user = $userRepo->find(3);

$participations = $user->getParticipations();

foreach ($participations as $participation) {
    echo $participation;
    $choices = $participation->getChoices();
    foreach ($choices as $choice) {
        echo "- ", $choice;
        $answers = $choice->getAnswers();
        echo "-- ", $choice->getQuestion();
        foreach ($answers as $answer) {
            echo "--- ", $answer;
        }
    }
}
```

```txt
Participation (Id: 1, User (id: 3, firstname: First 2, lastname: LAST 2, role: user, address: )
, Poll (id: 1, title: Doctrine 2, ça vous dit ?, created: 2017-03-03T08:00:00+0000)
)
- Choice (id: 1)
-- Question (id: 2, wording: Doctrine 2 est-il un bon ORM ?)
--- Answer (id: 3, wording: Oui, bien sûr !)
- Choice (id: 2)
-- Question (id: 3, wording: Doctrine 2 est-il facile d'utilisation ?)
--- Answer (id: 5, wording: Oui, il y a une bonne documentation !)
--- Answer (id: 6, wording: Oui, il y a de bons tutoriels !)
```

Voilà donc un exemple d'implémentation qui permet d'avoir les participations de chaque utilisateur.

Le schéma complet de notre base de données ressemble maintenant à :

![Schéma complet de la base de données](https://zestedesavoir.com/media/galleries/3902/f38ccd28-bd87-446c-a8cb-e586be916c84.png)