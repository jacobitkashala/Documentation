Le principe des relations `ManyToOne` reste semblable à une relation `OneToOne`. *Doctrine* utilise la notion de collection afin de bien gérer nos entités.

Nous pouvons donc exploiter l'[API de cette classe](https://github.com/doctrine/collections/blob/master/lib/Doctrine/Common/Collections/ArrayCollection.php) pour consulter, ajouter, modifier ou supprimer une entité appartenant à une relation 1..n très facilement.

Notre application reste ainsi cohérente et facile d'utilisation.

[[attention]]
| Dans les faits, une annotation `OneToMany` ne peut pas exister toute seule. Si la relation 1..n est unidirectionnelle, il faut obligatoirement avoir l'annotation `ManyToOne`. C'est une contrainte que *Doctrine* nous impose (Cf Annexe).