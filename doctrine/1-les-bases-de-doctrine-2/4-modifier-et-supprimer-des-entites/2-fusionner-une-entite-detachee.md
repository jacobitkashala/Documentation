Nous avons la possibilité de gérer une entité qui n'est pas issue de la base de données sans passer la méthode `persist`.
Prenons l'exemple classique où dans la session d'une application web, nous avons sérialisé un objet utilisateur.

Si nous dé-sérialisons ce même objet, il ne sera plus géré par *Doctrine*. Mais en utilisant la méthode `merge` de l'*entity manager*, l'entité est considérée comme si elle provenait de la base de données.

```php
<?php
# update-user.php

$entityManager = require_once join(DIRECTORY_SEPARATOR, [__DIR__, 'bootstrap.php']);

use Tuto\Entity\User;

$user = unserialize("informations de l'utilisateur serialisées");

// Reprise en charge de l'objet par l'entity manager
$entityManager->merge($user);

$user->setFirstname("First Real Modification");

$entityManager->flush();
```

Le prénom de l'utilisateur sera mis à jour avec cette méthode. 

La méthode `merge` ne doit être utilisée que dans un cadre où vous avez **la certitude** que les informations contenues dans l'objet sont déjà en parfaite synchronisation avec ceux dans la base de données. Sinon vous risquez d'écraser des modifications faites sur l'entité entre sa sérialisation et sa dé-sérialisation.