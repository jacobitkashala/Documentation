# Énoncé

Lorsqu'un utilisateur participe à un sondage, il choisit une réponse pour chacune des questions de celui-ci. 

Notre objectif est de stocker tous les choix que l'utilisateur a faits pour chacune de ses participations aux sondages et de pouvoir retrouver rapidement ces choix à partir d'un utilisateur.

Pour élargir le système de sondage, nous considérons aussi qu'il est possible d'avoir des questions avec plusieurs réponses possibles (question à choix multiple).

# Indices

- Les choix des utilisateurs doivent être matérialisés par une entité.
- Un choix fait intervenir une question et une ou plusieurs réponses.
- Les participations des utilisateurs sont matérialisées grâce à l'entité *participation*. Il faut donc la mettre à jour pour rajouter les choix.
