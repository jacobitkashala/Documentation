
Puisque appeler la méthode `persist` permet de gérer une entité. Il est tentant de créer une nouvelle entité avec les informations à jour (l'identifiant restant le même) puis de sauvegarder celle-ci avec `persist`.

Mais comme vous vous en doutez, cela ne fera pas une mise à jour. Et pour en avoir le cœur net, nous allons effectuer cette manipulation.

```php
<?php
# update-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$index = 10;

$user = new User();
$user->setId($index + 1);
$user->setFirstname("First Modified ".$index);
$user->setLastname("LAST Modified ".$index);
$user->setRole("user");

$entityManager->persist($user);

$entityManager->flush();
```

![Mise à jour de l'utilisateur en echec](https://zestedesavoir.com/media/galleries/3902/6d36f2e5-2e38-47a9-ab52-05a8fee098aa.png)

En consultant la base de données, nous pouvons voir d'une nouvelle ligne a été crée dans la table.

[[erreur]]
| Dès que nous appelons la méthode `persist` sur une entité **pas encore gérée** par *Doctrine*, à l'appel de la méthode `flush`, une nouvelle entrée sera forcément créer dans la base de données.

Donc comme pour la suppression d'une entité, nous allons d'abord commencer par la récupérer depuis le *repository*. Ensuite la mise à jour de cette entité se résume juste à modifier ses attributs puis à *flusher*.

```php
<?php
# update-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$identifiant = 11;

$userRepo = $entityManager->getRepository(User::class);

// Récupération de l'utilisateur (donc automatiquement géré par Doctrine)
$user = $userRepo->find($identifiant);

$user->setFirstname("First Real Modification");
$user->setLastname("Last Real Modification");

$entityManager->flush();
```

![Mise à jour de l'utilisateur avec comme identifiant : 11](https://zestedesavoir.com/media/galleries/3902/5248d521-9c9b-4574-a7b4-aa3bbe821c14.png)

Le comportement de *Doctrine* est très grandement influencé par l'état des entités. Il faut donc avant chaque mise à jour de celle-ci s'assurer que l'entité est bien gérée par *Doctrine*.