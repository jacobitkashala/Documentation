Notre système de sondage est presque complet. Il ne reste plus qu'à gérer les participations des utilisateurs et notre modèle sera finalisé.

Sachant qu'un utilisateur peut participer à **plusieurs** sondages différents et qu'un sondage peut avoir **plusieurs** participations, nous avons là une relation n..m.

Ce genre de relations est, sans doute, le type de relation le plus complexe mais hélas assez courant.

*Doctrine* les gère avec l'annotation `ManyToMany`. Comme pour les autres types de relation, une relation `ManyToMany` peut être bidirectionnelle. La configuration étant semblable, nous ne l'aborderons pas.