MySQL est une base de données relationnelle (tout comme MariaDB, PostgreSQL, Oracle, etc.) et il n'est pas rare d'avoir des relations diverses lorsque nous modélisons une application.

Par exemple, un sondage est constitué d'un ensemble de questions, une question peut avoir plusieurs réponses possibles, un utilisateur peut avoir une adresse, etc.

Toutes ces relations peuvent être matérialisées en utilisant des clés étrangères en SQL.

Avec *Doctrine*, selon la nature de la relation, nous avons des moyens très **simples**, **efficaces** et **élégants** de la gérer. Nous allons donc étoffer notre modèle de données en implémentant des relations avec *Doctrine*.

