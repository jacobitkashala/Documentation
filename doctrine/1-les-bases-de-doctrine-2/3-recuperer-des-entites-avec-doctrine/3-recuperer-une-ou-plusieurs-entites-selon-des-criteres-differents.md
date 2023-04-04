L'intérêt de *Doctrine* ne se limite pas qu'à la récupération d'une entité en se basant sur la clé primaire. Grâce aux métadonnées que nous avons rajoutées à notre entité, nous sommes maintenant en mesure de faire plusieurs recherches sur les utilisateurs en utilisant **tous ses attributs**.

Nous pouvons nativement faire des recherches sur le nom, le prénom ou encore sur deux ou plusieurs champs en même temps et ce, sans écrire aucune ligne de code personnalisée.

# Les *repositories*

Les *repositories* (entrepôts) sont des classes spécialisées qui nous permettent de récupérer nos entités. Chaque *repository* permet ainsi d'interagir avec un type d'entité et faire la liaison entre notre code et la base de données.

Pour, par exemple, lire les informations sur les utilisateurs, nous aurons un *repository* dédié à l'entité *utilisateur*.
C'est une bonne pratique de travailler avec les *repositories* car elle facilite grandement les interactions avec nos entités.

En utilisant un *repository* pour les utilisateurs, notre exemple précédent devient :

```php
<?php
# get-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$userRepo = $entityManager->getRepository(User::class);

$user = $userRepo->find(1);

echo $user;
```

L'*entity manager* reste l'élément central et c'est grâce à lui que nous récupérons notre *repository*. Avec la méthode `find`, nous ne sommes plus obligés de préciser la classe de l'entité que nous cherchons car le *repository* ne gère qu'une seule classe : ici les utilisateurs.

# Les méthodes du *repository*

## Les méthodes find, findAll, findBy et findOneBy

En accédant au *repository*, nous disposons de plusieurs méthodes *simples* permettant de lire des données.

Voici un petit tableau descriptif de chacune de ses méthodes *simples*.

->

| Méthode | Description  |
| --------| ------------------------------------- |
|  find   | Récupère une entité grâce à sa clé primaire |
| findAll | Récupère toutes les entités |
| findBy  | Récupère une liste d'entités selon un ensemble de critères |
| findOneBy | Récupère une entité selon un ensemble de critères |

<-


Pour tester toutes ces méthodes, nous allons d'abord créer plusieurs utilisateurs.

```php
<?php
# create-users.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

foreach (range(1, 10) as $index) {
    $user = new User();
    $user->setFirstname("First ".$index);
    $user->setLastname("LAST ".$index);
    $user->setRole("user");
    $entityManager->persist($user);
}

$entityManager->flush();
```

![Jeu de test pour la lecture](https://zestedesavoir.com/media/galleries/3902/775f158c-4dbc-49c0-a787-7798251337ea.png)

[[information]]
| Nous ne sommes pas obligés d'appeler la méthode `flush` après chaque `persist`. Dans l'extrait de code, nous donnons d'abord à *Doctrine* toutes les entités à sauvegarder avant de faire le `flush`.

Voici maintenant un ensemble d'exemples de requêtes de lecture : 

```php
<?php
# get-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$userRepo = $entityManager->getRepository(User::class);

$user = $userRepo->find(1);
echo "User by primary key:\n";
echo $user;

$allUsers = $userRepo->findAll();
echo "All users:\n";
foreach ($allUsers as $user) {
    echo $user;
}

$usersByRole = $userRepo->findBy(["role" => "admin"]);
echo "Users by role:\n";
foreach ($usersByRole as $user) {
    echo $user;
}

$usersByRoleAndFirstname = $userRepo->findBy(["role" => "user", "firstname" => "First 2"]);
echo "Users by role and firstname:\n";
foreach ($usersByRoleAndFirstname as $user) {
    echo $user;
}
```

Le résultat est assez explicite :
```txt
User by primary key:
User (id: 1, firstname: First, lastname: LAST, role: admin)

All users:
User (id: 1, firstname: First, lastname: LAST, role: admin)
User (id: 2, firstname: First 1, lastname: LAST 1, role: user)
User (id: 3, firstname: First 2, lastname: LAST 2, role: user)
User (id: 4, firstname: First 3, lastname: LAST 3, role: user)
User (id: 5, firstname: First 4, lastname: LAST 4, role: user)
User (id: 6, firstname: First 5, lastname: LAST 5, role: user)
User (id: 7, firstname: First 6, lastname: LAST 6, role: user)
User (id: 8, firstname: First 7, lastname: LAST 7, role: user)
User (id: 9, firstname: First 8, lastname: LAST 8, role: user)
User (id: 10, firstname: First 9, lastname: LAST 9, role: user)
User (id: 11, firstname: First 10, lastname: LAST 10, role: user)

Users by role:
User (id: 1, firstname: First, lastname: LAST, role: admin)

Users by role and firstname:
User (id: 3, firstname: First 2, lastname: LAST 2, role: user)
```

Les méthodes les plus intéressantes ici sont le `findBy` et le `findOneBy`. Elles prennent toutes les deux un tableau associatif avec comme clé le nom des attributs de notre entité (id, firstname, lastname, role) et comme valeur l'information que nous voulons chercher.

[[erreur]]
| Nous devons toujours mettre le nom des attributs des entités dans nos filtres et pas celui des colonnes dans la base de données. En utilisant l'ORM, nous devons faire fi de la configuration réelle de la base de données. *Doctrine* s'occupe en interne de la correspondance entre un attribut de notre classe et une colonne de la base de données.

 Si nous avons deux ou plusieurs clés dans le tableau comme `["role" => "user", "firstname" => "First 2"]`, *Doctrine* va chercher tous les utilisateurs ayant comme rôle `user` **et** comme prénom `First 2`.

[[question]]
| Du coup, comment faire pour récupérer les utilisateurs ayant comme rôle `user` **ou** un prénom valant `First 2` ?

Il existe d'autres moyens pour récupérer des données avec des critères beaucoup plus avancés que ceux proposés nativement par le *repository*. Nous aurons l'occasion de tous les voir dans la suite de ce cours.

Mais avant cela, nous pouvons tester les filtres natifs comme *Order By*, *Limit*, *Offset* qui permettent d'affiner un tant soit peu les résultats de nos requêtes.

```php
<?php
# get-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$userRepo = $entityManager->getRepository(User::class);

$limit = 4;
$offset = 2;
$orderBy = ["firstname" => "DESC"];
$usersByRoleWithFilters = $userRepo->findBy(["role" => "user"], $orderBy, $limit, $offset);
echo "Users by role with filters:\n";
foreach ($usersByRoleWithFilters as $user) {
    echo $user;
}
```

Avec cette requête, nous récupérons une liste d'utilisateur en les triant par ordre décroissant du prénom (`["firstname" => "DESC"]`) et en limitant le nombre de résultats grâce à l'offset (`2`) et la limite (`4`).

Le résultat d'une telle requête est :

```txt
Users by role with filters:
User (id: 8, firstname: First 7, lastname: LAST 7, role: user)
User (id: 7, firstname: First 6, lastname: LAST 6, role: user)
User (id: 6, firstname: First 5, lastname: LAST 5, role: user)
User (id: 5, firstname: First 4, lastname: LAST 4, role: user)
```

Le filtre *Order By* peut avoir comme valeur `ASC` pour un trie par ordre croissant et `DESC` pour un trie par ordre décroissant.

## Les méthodes findByXXX et findOneByXXX

En plus des méthodes simples `findBy` et `findOneBy`, nous avons une panoplie de méthodes qui permettent de faire une recherche.

Ce sont des raccourcis qui rajoutent du sucre syntaxique aux méthodes simples mais restent néanmoins moins complets.

Pour chacune de nos entités, *Doctrine* gère grâce aux méthodes magiques plusieurs variantes de méthodes de sélection.

Si nous prenons le cas de notre utilisateur, l'attribut `role` permet d'avoir ainsi deux méthodes dans le *repository*: `findByRole` et `findOneByRole`. 
Ces deux méthodes représentent respectivement les méthodes `findBy(["role" => "XXX"])` et `findOneBy(["role" => "XXX"])`.

Donc l'exemple précédent peut devenir :

```php
<?php
# get-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$userRepo = $entityManager->getRepository(User::class);

//$usersByRole = $userRepo->findBy(["role" => "admin"]);
$usersByRole = $userRepo->findByRole("admin");
echo "Users by role:\n";
foreach ($usersByRole as $user) {
    echo $user;
}
```

[[i]]
|Ces méthodes magiques sont moins complètes car elles ne supportent pas les filtres natifs (*Order By*, *Limit*, etc.).