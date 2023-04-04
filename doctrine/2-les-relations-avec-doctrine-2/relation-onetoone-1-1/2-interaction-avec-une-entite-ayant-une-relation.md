
Essayons maintenant d'affecter une adresse à un de nos utilisateurs :

```php
<?php
# set-user-address.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;
use Tuto\Entity\Address;

$userRepo = $entityManager->getRepository(User::class);

$user = $userRepo->find(1);

$address = new Address();
$address->setStreet("Champ de Mars, 5 Avenue Anatole");
$address->setCity("Paris");
$address->setCountry("France");

$user->setAddress($address);

$entityManager->flush();

```

En exécutant le code, une belle erreur s'affiche :

```txt
Fatal error: Uncaught Doctrine\ORM\ORMInvalidArgumentException: 
A new entity was found through the relationship 'Tuto\Entity\User#address' that was not configured to cascade 
persist operations for entity: Address (id: , street: Champ de Mars, 5 Avenue Anatole, city: Paris, country: France). 
To solve this issue: Either explicitly call EntityManager#persist() on this unknown entity or 
configure cascade persist  this association in the mapping for example @ManyToOne(..,cascade={"persist"}).
```

Vous l'aurez deviné, l'entité *adresse* n'est pas encore gérée par *Doctrine*. Notre ORM ne sait donc pas quoi faire avec. Pour résoudre ce problème nous avons deux solutions que nous allons aborder ci-dessous.

# Gestion manuelle des relations

Comme le message d'erreur le suggère, nous pouvons corriger le problème en utilisant directement la méthode `persist` sur l'entité *adresse* avant de flusher. Elle sera ainsi gérée par *Doctrine*. Le code devient alors :

```php
<?php
# set-user-address.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;
use Tuto\Entity\Address;

$userRepo = $entityManager->getRepository(User::class);

$user = $userRepo->find(1);

$address = new Address();
$address->setStreet("Champ de Mars, 5 Avenue Anatole");
$address->setCity("Paris");
$address->setCountry("France");

$user->setAddress($address);

$entityManager->persist($address);
$entityManager->flush();
```

En ré-exécutant le code, une adresse est créée et affectée à notre utilisateur.

![Adresse sauvegardée](https://zestedesavoir.com/media/galleries/3902/c83f5fed-fd2b-4d9e-9074-b4b1cc567894.png)

![Utilisateur avec une adresse](https://zestedesavoir.com/media/galleries/3902/10d22365-8430-49a3-a8e1-d6fa86e84c78.png)

# Délégation à Doctrine

Nous pouvons aussi utiliser le système de cascade de *Doctrine* pour gérer la relation entre les entités *utilisateur* et *adresse*. 
Avec ce système, nous pouvons définir comment *Doctrine* gère l'entité *adresse* si l'entité *utilisateur* est modifiée.
Ainsi, pendant la création et la suppression de l'utilisateur, nous pouvons demander à *Doctrine* de répercuter ces changements sur son adresse.

Le paramètre à utiliser est l'attribut `cascade` de l'annotation `OneToOne`. Voici un tableau récapitulatif de quelques valeurs possibles et de leurs effets.

->

| Cascade | Effet  |
| ------- | -------------------------------------------------------------------|
| persist | Si l'entité *utilisateur* est sauvegardée, faire de même avec l'entité adresse associée|
| remove | Si l'entité *utilisateur* est supprimée, faire de même avec l'entité adresse associée|

<-

[[attention]]
| Il faut porter une attention particulière lors de l'utilisation de la cascade d'opérations. Les performances de votre application peuvent en pâtir si cela est mal configurée.

Pour notre cas, puisqu'une adresse ne sera associée qu'à un et un seul utilisateur et vice versa, nous pouvons nous permettre de la créer ou de la supprimer suivant l'état de l'entité *utilisateur*.

L'entité *utilisateur* devient :

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
* @ORM\Entity
* @ORM\Table(name="users")
*/
class User
{
    // ...

    /**
    * @ORM\OneToOne(targetEntity=Address::class, cascade={"persist", "remove"})
    */
    protected $address;

   // ...
}
```

N'hésitez pas à valider les annotations en utilisant les commandes que *Doctrine* met à notre disposition.
```txt
vendor/bin/doctrine orm:validate-schema               
[Mapping]  OK - The mapping files are correct.
[Database] OK - The database schema is in sync with the mapping files.
```

[[attention]]
| Avec l'attribut `cascade`, les opérations de cascade sont effectuées par *Doctrine* au niveau applicatif. Les événements SQL de type `ON DELETE` et `ON UPDATE` sur les clés étrangères ne sont pas utilisés.