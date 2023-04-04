
Maintenant que notre entité est bien configurée et que la base de données est à jour, nous pouvons utiliser l'ORM pour la manipuler.

Nous avons à notre disposition un objet que vous avez sûrement remarqué durant l'étape d'installation et de configuration de *Doctrine* : le gestionnaire d'entités (entity manager).

Comme son nom l'indique, cet objet est le chef d'orchestre de l'ORM *Doctrine*. Toutes nos opérations sur les entités se feront directement ou indirectement par son intermédiaire.

Nous l'appellerons durant le reste du cours avec son nom anglais : *entity manager*.

Pour créer notre premier utilisateur, nous allons créer un fichier ***create-user.php***. Avec *Doctrine*, la création d'une entité se réduit à deux actions :

- l'instanciation de l'entité ;
- la persistance de celle-ci grâce à l'*entity manager*.

Voyez par vous-même.

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

// Vérification du résultats
echo "Identifiant de l'utilisateur créé : ", $admin->getId();
```

Si nous exécutons ce code, nous obtenons comme résultat :

```text
Identifiant de l'utilisateur créé : 1
```

![Création du premier utilisateur](https://zestedesavoir.com/media/galleries/3902/57f3d796-f591-4688-ae2c-2cc6d63b0863.png)

En utilisant la méthode `persist` de l'*entity manager*, nous disons à *Doctrine* qu'il peut planifier la sauvegarde de cet objet. Mais à ce stade, aucune requête SQL n'est exécutée.

C'est à l'appel de la méthode `flush` que *Doctrine* effectue la sauvegarde réelle de l'entité. Nous verrons plus tard l'intérêt de l'utilisation de ces deux méthodes distinctes.

Un dernier point intéressant à relever aussi est l'affectation automatique d'un identifiant à notre utilisateur. En effet, grâce à *Doctrine*, l'identifiant SQL auto-incrémenté est récupéré dès qu'on fait appel à la méthode `flush`.
Notre entité se retrouve donc tout le temps en parfaite synchronisation avec la base de données.