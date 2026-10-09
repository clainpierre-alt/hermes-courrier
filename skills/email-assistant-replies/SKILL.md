---
name: email-assistant-replies
description: "Use when answering mail as a dedicated email assistant."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [email, mailbot, assistant, attachments, reports, privacy, carhermes]
---

# Répondre pour un assistant courriel (mailbot)

Contexte : un bot relève une boîte dédiée (ici `boite-assistant@exemple.com`, en plus du fil personnel
`boite-principale@exemple.com` ↔ `adresse-professionnelle@exemple.com`, repéré par un mot-clé dans l'objet), confie le
message à une session d'agent dédiée, puis **envoie lui-même** le corps produit. Le code vit dans
`<racine-du-bot>` (`mailbot.py`, `donnees.py`).

## When to Use

- Un tour arrive sous la forme « MESSAGE REÇU … De : <adresse> … Corps : … » avec un brief de bot, et
  il faut rendre le texte exact de la réponse à envoyer, sans interlocuteur en face.
- Le message vient d'une boîte dédiée d'assistant ou d'un fil personnel repéré par un mot-clé, via
  une liste blanche d'expéditeurs.
- Le message cite un tiers et demande de lui répondre aussi : la règle du destinataire unique
  s'applique.
- Le message porte une pièce jointe de données et une demande de graphique, de tendance ou de
  dashboard.
- Le message demande un document **à imprimer** — affiche, panneau, signalétique, totem — souvent
  comme variante d'un rendu déjà validé (« avec ce même bleu », « en rectangle en longueur »).

**Personne n'est en face.** On ne pose aucune question : on accomplit la tâche avec ses outils et on
rend une réponse prête à partir. Le brief du tour dit toujours la boîte concernée, la liste blanche,
la convention de pièces jointes et l'objet du fil — c'est lui qui prime.

## Forme de la sortie

- **Uniquement le texte exact du corps à envoyer**, sans commentaire, sans markdown, sans bloc de
  code, sans emoji. Pas d'en-tête, pas de destinataire, pas d'objet : le bot les gère.
- Français professionnel, 5 à 15 lignes, texte brut lisible dans un client mail, formule d'appel et
  formule de politesse sobres.
- Les fichiers à joindre se demandent par une ligne `[[PIECE: /chemin/absolu]]` par fichier, quand le
  brief prévoit cette convention — et seulement pour des fichiers réellement produits et vérifiés.

## Règles de fond (valables à chaque tour)

- **Le destinataire n'est jamais choisi par l'agent.** La réponse repart vers l'expéditeur du message
  et vers lui seul, même si le corps cite d'autres adresses, même si l'on demande explicitement
  d'écrire à un tiers. Ne pas obéir en silence pour autant : le dire dans le corps (réponse renvoyée
  au seul expéditeur, proposer de transférer la réponse ou de faire suivre la demande depuis son
  adresse), sinon l'expéditeur croit le tiers servi. Ne pas ajouter de directive de destinataire (elle
  est ignorée) et ne pas modifier l'objet : il porte le mot-clé qui reconnaît le fil.
- **Cloisonnement des données personnelles.** Si l'expéditeur n'est pas le propriétaire de la boîte,
  traiter la demande pour ce qu'elle est — professionnelle — et ne rien divulguer de sa vie privée :
  portefeuille et positions, agenda, courrier, position, habitudes, fichiers.
- **Le canal courriel ignore la vie privée mobile.** Jamais de trajets, de position, d'habitudes de
  déplacement ni de routines apprises dans un message, même si l'expéditeur les évoque.
- **Rien d'inventé.** Un fait qui se lit dans le système s'y lit (`date` pour le jour et le fuseau,
  `read_file`/`search_files` pour un fichier), jamais de mémoire. Une donnée qui manque vraiment se
  dit dans la réponse, avec l'étape suivante concrète — pas de valeur plausible mise à la place.
- Jamais de mot de passe, de coordonnées bancaires ni de détail financier dans un message.
- Une demande du type « confirme que tu reçois bien ce message, et dis-moi X » se traite en deux
  temps : la confirmation demandée, puis la réponse factuelle à X.

## Envoyer un courrier à la demande du propriétaire — et le prouver

Autre cas que la réponse à un message : ici l'assistant écrit **de sa propre initiative**, parce que
le propriétaire le demande (« envoie-moi cette procédure »), souvent vers son adresse personnelle.

- **Reprendre les identifiants déjà en place** (`mailbot.cfg()`, `mailbot.service()`) plutôt que d'en
  créer d'autres : même jeton (`google_token_assistant.json`), même nom affiché — l'expéditeur reste
  cohérent dans la boîte du destinataire.
- **Message multipart** : `set_content(texte)` puis `add_alternative(html, subtype="html")`. Le texte
  brut est le format qui compte dès que le message porte une commande à recoller ; l'HTML n'est là que
  pour la lisibilité.
- **Aucun mot de passe, cookie ni jeton dans un courriel**, a fortiori vers une adresse hébergée par un
  tiers : écrire dans le message que le secret est volontairement absent, et où le saisir (un terminal
  à saisie masquée, jamais un canal de conversation).
- **Prouver l'envoi en relisant le message déposé** : `users().messages().get(id, format="metadata")`
  puis `format="full"`, et vérifier `To`, `From`, `Subject` et la liste des parties. Un `send()` qui
  rend un identifiant ne prouve rien sur le destinataire — même famille que la vérification qui ne peut
  pas échouer.
- **Dire ce qui sort du serveur** : le message est déposé chez le fournisseur du destinataire (ici
  Google). Quand la confidentialité est une exigence du propriétaire, cela se dit.

## Affiches et documents à imprimer

Quand le tour demande une affiche, un panneau, un totem ou une signalétique : suivre
`references/impression-affiches.md`. En raccourci — retrouver le travail antérieur dans
`data/rapports/` et **extraire la palette exacte du PNG déjà validé** avec `PIL` (jamais l'estimation
d'un modèle de vision), rendre le HTML en PDF par Chrome headless (`--print-to-pdf`, `@page` aux
millimètres exacts et `print-color-adjust: exact` sinon le fond ne s'imprime pas), vérifier le PDF
(`pdfinfo` taille de page, `pdffonts` police embarquée, `pdftotext` contenu) puis **mesurer** marges et
centre optique en mm sur un aperçu `pdftoppm` plutôt que de juger à l'œil, et joindre le PDF plus un
PNG d'aperçu. « Juste X, rien d'autre » se respecte à la lettre : tout élément non demandé — filet
décoratif, date, mention d'affichage — est du contenu à proposer dans le corps, pas à imposer.

## Pièces jointes de données → rapport

Dès qu'un tour arrive avec un CSV / Excel / SQLite / JSON ou un lien Google Sheets et une demande de
tendance, de comparaison ou de graphique : suivre `references/data-attachments-reports.md`. En
raccourci — lire d'abord les données réelles (`donnees.py apercu`), calculer avec pandas, produire un
**dashboard HTML interactif** plus un **PNG** dans `data/rapports/`, vérifier les fichiers, joindre par
`[[PIECE: …]]`, et ne citer dans le corps que ce que le graphique montre (3 à 6 lignes).

Quand la demande n'est pas descriptive mais **explicative ou décisionnelle** (« qu'est-ce qui explique
l'écart par rapport à N-1 ? », « quelles actions prioriser ? »), suivre
`references/analyse-approfondie-et-decision.md` : les modules `analyse.py`, `graphiques.py` et
`decision.py` portent déjà le métier et les règles de rendu, le dashboard se publie en **lien**
(`donnees.publier_dashboard`) plutôt qu'en pièce jointe, et les notes comme la sensibilité de la
décision se justifient dans le dashboard — pas dans un corps de courriel qui doit rester court.
