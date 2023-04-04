Pour bien voir, l'utilité de la notion d'entité gérée par *Doctrine*, nous allons essayer de supprimer l'utilisateur avec comme identifiant deux (2).

L'*entity manager* (oui encore lui) propose une méthode `remove` qui prend en paramètre l'entité que nous voulons supprimer.

```php
<?php
# delete-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$index = 1;

$user = new User();
$user->setId($index + 1);
$user->setFirstname("First ".$index);
$user->setLastname("LAST ".$index);
$user->setRole("user");

$entityManager->remove($user);
```

On a beau mettre l'identifiant et toutes les informations de l'entité que nous voulons supprimer, en exécutant le code, nous obtenons une exception.

```bash
Fatal error: Uncaught Doctrine\ORM\ORMInvalidArgumentException:
 Detached entity User (id: 2, firstname: First 1, lastname: LAST 1, role: user)
 cannot be removed
```

L'entité est considérée comme *détachée*. *Doctrine* ne le gère pas et donc ne sait pas comment la supprimer. C'est une erreur assez courante et qui peut, sur de grand projet, facilement vous faire perdre beaucoup de temps.

Pour supprimer l'entité, nous allons donc commencer par le récupérer grâce au *repository*.

```php
<?php
# delete-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$identifiant = 2;

$userRepo = $entityManager->getRepository(User::class);

// Récupération de l'utilisateur (donc automatiquement géré par Doctrine)
$user = $userRepo->find($identifiant);

$entityManager->remove($user);
$entityManager->flush($user);

// Récupération pour vérifier la suppression effective de l'utilisateur
$user = $userRepo->find($identifiant);

var_dump($user); // doit renvoyer NULL
```

![Suppression de l'utilisateur](https://zestedesavoir.com/media/galleries/3902/e796339b-3ff0-4a29-823d-50c2e31a51a7.png)

Comme pour la création d'une entité, nous disons d'abord à *Doctrine* que nous souhaitons supprimer une entité, et ensuite nous validons l'opération en appelant la méthode `flush`.