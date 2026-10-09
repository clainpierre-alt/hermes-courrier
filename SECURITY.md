# Politique de sécurité

## Signaler un problème

Ce dépôt est un exemple d'implémentation et une compétence d'agent. S'il contient, par inadvertance,
un secret, une adresse personnelle ou un jeton, **ne pas ouvrir d'issue publique** : le signaler par
un canal privé à la personne qui vous l'a transmis, afin qu'il soit retiré puis que l'historique soit
réécrit si nécessaire.

## Ce qui ne doit jamais entrer dans ce dépôt

- des **jetons** d'accès (Google, IMAP, SMTP, Telegram, API diverses) ;
- des **fichiers de session** ou de *credentials* ;
- des **mots de passe d'application** ;
- des **adresses électroniques ou numéros de téléphone réels** de personnes ;
- des **URL d'administration** d'un serveur privé ;
- des **journaux** de production.

Le fichier `.gitignore` les exclut par motif, mais un `.gitignore` ne protège que ce qui n'a pas déjà
été ajouté de force : vérifier avant chaque envoi.

## Modèle de menace de l'implémentation de référence

| Risque | Mitigation dans le code |
|---|---|
| Un tiers écrit à la boîte et fait agir l'agent | Liste blanche d'expéditeurs stricte ; tout le reste est ignoré |
| Deux réponses pour un même message | Verrou de fichier + état persistant par identifiant de message |
| Réponse à une adresse automatique | Refus explicite des motifs `no-reply`, `noreply`, `donotreply` |
| Envoi massif après une erreur de boucle | Plafond par passe |
| Envoi involontaire pendant la mise au point | Mode à blanc `--dry-run` (n'envoie pas, ne marque pas) |
| Jeton laissé en clair dans le dépôt | Le jeton est désigné par la configuration, jamais écrit dans le code |

## Périmètre

Ce dépôt ne s'occupe pas de la sécurité de la boîte elle-même (authentification à deux facteurs,
révocation des sessions, chiffrement au repos) : ces sujets relèvent de votre fournisseur de
messagerie.
