
Pour accéder aux informations grâce à une clé primaire, nous devons juste renseigner deux paramètres :

- la classe de l'entité que nous voulons récupérer ;
- et l'identifiant de celle-ci.

L'*entity manager* se charge alors de faire la requête SQL et d'instancier notre classe. Un exemple valant mieux que mille discours, nous allons récupérer l'utilisateur que nous venons de créer.

```php
<?php
# get-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$user = $entityManager->find(User::class, 1);

echo sprintf(
    "User (id: %s, firstname: %s, lastname: %s, role: %s)", 
    $user->getId(), $user->getFirstname(), $user->getLastname(), $user->getRole()
);
```

En exécutant ce code, nous obtenons comme réponse :

```txt
User (id: 1, firstname: First, lastname: LAST, role: admin)
```

[[information]]
| Si l'identifiant n'existe pas, la variable `$user` vaudra `null`.

Pour la suite, nous allons rajouter une méthode `__toString` à nos entités pour pouvoir les afficher plus facilement. Vous pouvez donc dés à présent rajouter dans l'entité *utilisateur*, ce code :

```php
<?php
public function __toString()
{
    $format = "User (id: %s, firstname: %s, lastname: %s, role: %s)\n";
    return sprintf($format, $this->id, $this->firstname, $this->lastname, $this->role);
}
```