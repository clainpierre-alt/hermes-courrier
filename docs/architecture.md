# Architecture — pourquoi le bot est fait ainsi

Ce document explique les décisions de conception de `src/mailbot.py`. Le code est court ; ce sont les
raisons qui coûtent cher à redécouvrir.

## 1. Le bot envoie, l'agent rédige

Un agent qui aurait un accès direct au courrier pourrait écrire n'importe quoi à n'importe qui, sans
trace. Ici, l'agent **produit un texte** et le bot l'adresse et l'envoie. La chaîne d'envoi reste du
code déterministe : on peut l'auditer, la plafonner, la mettre à blanc.

Conséquence pratique : l'agent n'a pas besoin d'outil de messagerie, seulement de produire le corps de
la réponse. Le contrat est décrit dans la compétence.

## 2. Idempotence — la partie qui casse tout si on l'oublie

Un tour d'agent dure **plusieurs minutes** ; le minuteur se déclenche toutes les cinq minutes. Sans
protection, deux passes lisent le même message non lu avant que la première ne l'ait marqué : 

- **double réponse** à l'utilisateur ;
- et parfois double action (deux fichiers déposés, deux appels).

Deux mécanismes, complémentaires :

- **un verrou de fichier** (`flock`) : une seule passe à la fois ;
- **un état persistant par identifiant de message** : un message déjà traité ne l'est plus jamais,
  même si le marquage a échoué.

Le verrou protège contre la concurrence, l'état contre la répétition. Les deux sont nécessaires.

## 3. Filtres d'entrée : deux conditions, pas une

Un message n'est traité que si **les deux** sont vraies :

1. l'expéditeur est dans la **liste blanche** ;
2. l'**objet contient un mot-clé** convenu.

Une seule condition serait trop permissive : une liste blanche seule ferait traiter toute la
correspondance d'une personne autorisée ; un mot-clé seul ouvrirait le bot au premier venu.

## 4. Refus des adresses automatiques

Répondre à `no-reply@` crée des boucles et des rejets. Le bot refuse explicitement ces motifs plutôt
que de compter sur la chance.

## 5. Plafond par passe

Un bug de boucle ne doit pas envoyer cinquante messages avant qu'on s'en aperçoive. Le plafond borne
les dégâts par passe ; il se règle dans la configuration (`max_per_pass`).

## 6. Mode à blanc

`--dry-run` montre ce qui **serait** fait sans rien envoyer ni marquer. C'est le mode par défaut dans
le doute, et le seul à utiliser pendant la mise au point.

## 7. Configuration hors du code

Le code ne contient **aucune** adresse, aucun jeton, aucun chemin propre à une machine : tout vient de
`config.json`. C'est ce qui permet de publier ce dépôt sans rien exposer — et de le reprendre ailleurs
sans édition du code.

## 8. Ce que l'on n'a pas fait, et pourquoi

- **Pas de base de données.** Un fichier d'état suffit, se sauvegarde, et se lit à la main en cas de
  doute. Une base aurait été plus lourde sans rien apporter.
- **Pas d'envoi par l'agent.** Voir le point 1 : la chaîne d'envoi reste inspectable.
- **Pas de traitement parallèle des messages.** Le débit réel (quelques messages par jour) ne le
  justifie pas, et le parallélisme rouvrirait exactement le problème du point 2.
- **Pas de résumé automatique de l'historique.** L'état ne retient que ce qui est nécessaire à
  l'idempotence : moins il y a d'écrit, moins il y a à protéger.
