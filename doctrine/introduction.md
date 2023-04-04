
La programmation orientée objet a fini par prendre une part importante dans le monde de PHP. Bien qu'à priori simple, le besoin récurrent de faire communiquer un code orienté objet avec des sources de données extérieures telles que des base de données cache une grande complexité.

[[question]]
| Comment faire correspondre facilement nos objets PHP aux informations dans la base de données ? Comment maintenir la cohérence entre le modèle objet de l'application et le schéma de la base de données ?

Ces problématiques, communes à tous les langages, ont été résolues grâce au mapping objet-relation (*Object-Relational Mapping* ou ORM). Cette technique permet d'établir un lien étroit entre notre modèle de données et la base de données relationnelles et nous donne ainsi le sentiment d'avoir une base de données orientée objet.

[[information]]
| Le sigle ORM (*Object-Relational **Mapper***) est aussi utilisé pour désigner les librairies qui implémentent cette technique.

Dans le monde du PHP, plusieurs librairies permettent de remplir ce besoin. nous pouvons citer [Propel](http://propelorm.org) ou encore [Eloquent (L'ORM de Laravel)](https://laravel.com/docs/master/eloquent).

Et parmi toutes ces librairies, nous allons découvrir [Doctrine 2](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/) qui est très mature et largement supportée par presque tous les frameworks de l'écosystème de PHP (Symfony, Zend Framework, etc.).  Nous aborderons entre autres :

- comment installer et configurer *Doctrine 2* ;
- comment modéliser un système de données orienté objet avec *Doctrine 2* ;
- comment exploiter une base de données avec *Doctrine 2*.

Il est important de souligner que l’utilisation d’un ORM nécessite une bonne connaissance de la programmation orientée objet en PHP et des notions en modélisation avec des méthodes d’analyse et de conception comme UML[^uml], Merise, etc.

Pour suivre ce cours, il faudra aussi savoir installer et utiliser le gestionnaire de dépendances [Composer](https://getcomposer.org/) ainsi qu'un moteur de base de données relationnelle (MySQL, PostgreSQL, MSSQL, Oracle, etc.).

[^uml]: **U**nified **M**odeling **L**anguage - Langage de modélisation unifié