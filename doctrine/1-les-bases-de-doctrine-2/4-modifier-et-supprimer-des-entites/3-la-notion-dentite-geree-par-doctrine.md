
Reprenons l'exemple de création d'un utilisateur :

```php
<?php
# create-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

// Instanciation de l'utilisateur

$admin = new User();
$admin->setFirstname("First");
$admin->setLastname("LAST");
$admin->setRole("admin");

// Gestion de la persistance
$entityManager->persist($admin);
$entityManager->flush();

// Vérification du résultat
echo "Identifiant de l'utilisateur crée : ", $admin->getId();
```

Bien que l'entité *utilisateur* soit bien configurée avec les annotations de *Doctrine*, lorsque nous instancions un utilisateur, il n'y a rien qui lie l'instance elle-même à *Doctrine*.

Il faut bien faire la distinction entre la classe `User` et les instances de celle-ci. Les annotations sur la classe permettent de dire à *Doctrine* comment gérer des instances de ce type.

Par contre, tant que nous ne demandons pas à *Doctrine* de le faire, nos instances n'auront aucun lien avec notre ORM.

Donc avant l'appel à la méthode `persist` de l'*entity manager*, l'instance n'était pas liée à *Doctrine* et n'etait donc **pas gérée**.

Par contre, dès que nous récupérons une entité via l'*entity manager* ou le *repository* ou bien dès que la méthode `persist` est appelée, le ou les entités **sont gérées** par *Doctrine*.

Ainsi, *Doctrine* surveille les modifications faites sur ces entités et appliquent ces changements à l'appel de la méthode `flush`.

![Notion d'entité gérée par Doctrine](https://zestedesavoir.com/media/galleries/3902/d7c958c2-f567-4096-8443-4114d1d8f08a.png)
