
Nous allons tester notre configuration en créant une participation au sondage « Doctrine 2, ça vous dit ? ».

```php
<?php
# create-participation.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\Participation;
use Tuto\Entity\Poll;
use Tuto\Entity\User;

$participation = new Participation();

$participation->setDate(new \Datetime("2017-03-03T09:00:00Z"));

$pollRepo = $entityManager->getRepository(Poll::class);
$poll = $pollRepo->find(1);

$userRepo = $entityManager->getRepository(User::class);
$user = $userRepo->find(4);

$participation->setUser($user);
$participation->setPoll($poll);

$entityManager->persist($participation);
$entityManager->flush();

echo $participation;
```


```txt
Participation (Id: 1, User (id: 4, firstname: First 3, lastname: LAST 3, role: user, address: )
, Poll (id: 1, title: Doctrine 2, ça vous dit ?, created: 2017-03-03T08:00:00+0000)
)
```


[[erreur]]
| Grâce à la clé primaire que nous avons définie, un utilisateur ne pourra pas participer deux fois au même sondage. Si vous ré-exécuter l'extrait de code, vous aurez une erreur fatale (Fatal error: Uncaught PDOException: SQLSTATE[23000]: Integrity constraint violation: 1062 Duplicata du champ '4-1' pour la clef 'user_poll_unique').