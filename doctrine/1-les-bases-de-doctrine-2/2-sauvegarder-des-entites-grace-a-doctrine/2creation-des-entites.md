
Nous allons rentrer dans le vif du sujet en modélisant les utilisateurs qui participeront et créeront éventuellement les sondages.

Nous allons considérer qu'un utilisateur a un identifiant, un nom, un prénom et un rôle (administrateur, modérateur, etc.).

Cette classe simple peut représenter notre objet utilisateur.

```php
<?php
# src/Entity/User.php

namespace Tuto\Entity;

class User
{
    protected $id;

    protected $firstname;

    protected $lastname;

    protected $role;

    public function getId()
    {
        return $this->id;
    }
     
    public function setId($id)
    {
        $this->id = $id;
    }
     
    public function getFirstname()
    {
        return $this->firstname;
    }
     
    public function setFirstname($firstname)
    {
        $this->firstname = $firstname;
    }
     
    public function getLastname()
    {
        return $this->lastname;
    }
     
    public function setLastname($lastname)
    {
        $this->lastname = $lastname;
    }
     
    public function getRole()
    {
        return $this->role;
    }
     
    public function setRole($role)
    {
        $this->role = $role;
    }
}
```

À l'état actuel, nous n'avons qu'une classe PHP comme tant d'autres. Pour l'utiliser avec *Doctrine*, nous devons le décorer.