# Hermes Courrier — un bot qui répond au courriel

Un assistant qui **relève une boîte dédiée, confie chaque message à un agent, rédige la réponse et
l'envoie lui-même** — avec des garde-fous stricts : liste blanche d'expéditeurs, un seul envoi par
message, état persistant (idempotence), refus d'écrire aux adresses automatiques, plafond par passe,
mode à blanc.

Ce dépôt contient la **connaissance** (une compétence Hermes) et une **implémentation de référence**
(`src/mailbot.py`), tous deux assainis. Il est fait pour être repris par un autre agent : la compétence
est auto-suffisante, le code est un exemple à adapter.

## Sommaire

- [Comment ça marche](#comment-ça-marche)
- [Ce qu'il y a dans ce dépôt](#ce-quil-y-a-dans-ce-dépôt)
- [Reprendre la compétence](#reprendre-la-compétence)
- [Reprendre le code](#reprendre-le-code)
- [Ce qui a été retiré avant publication](#ce-qui-a-été-retiré-avant-publication)
- [Règles de sécurité](#règles-de-sécurité)

## Comment ça marche

```
boîte dédiée ──► relevé (IMAP/Gmail API) ──► filtre (expéditeur en liste blanche + mot-clé d'objet)
                                              │
                                              ▼
                                   l'agent rédige la réponse
                                              │
                                              ▼
                   envoi par le bot lui-même ──► marquage du message (idempotence)
```

Trois propriétés portent tout le reste :

1. **Le bot envoie, pas l'agent.** L'agent produit le *texte* de la réponse ; c'est le bot qui
   l'adresse et l'expédie. Un agent qui n'a pas d'accès direct au courrier ne peut pas écrire à
   l'insu de la chaîne.
2. **Idempotence.** Chaque tour est long (plusieurs minutes), alors que le minuteur se déclenche plus
   souvent. Sans verrou ni état persistant, deux passes lisent le même message non lu et
   l'utilisateur reçoit **deux réponses**. Un verrou de fichier et un état par identifiant de message
   règlent le problème ; ce n'est pas un confort, c'est une nécessité.
3. **Liste blanche d'expéditeurs.** Le bot ne traite que les messages venant d'adresses autorisées.
   Tout le reste est ignoré, sans réponse.

## Ce qu'il y a dans ce dépôt

| Chemin | Rôle |
|---|---|
| `skills/email-assistant-replies/SKILL.md` | La compétence Hermes : quand l'utiliser, ce que l'agent doit produire, les pièges. |
| `src/mailbot.py` | L'implémentation de référence, assainie : relevé, filtres, rédaction, envoi, état. |
| `src/config.example.json` | Le fichier de configuration attendu, valeurs vides. |
| `docs/architecture.md` | Les décisions de conception et pourquoi elles ont été prises (guère de code, beaucoup de raisons). |

## Reprendre la compétence

Copier `skills/email-assistant-replies/` dans le dossier `skills/` de l'agent :

```bash
cp -r skills/email-assistant-replies "$HERMES_HOME/skills/email/"
```

La compétence est autonome : elle décrit le contrat (ce que l'agent reçoit, ce qu'il doit rendre) et
les pièges. Aucune dépendance à ce dépôt n'est nécessaire à l'exécution.

## Reprendre le code

```bash
cp src/config.example.json /chemin/vers/config.json   # puis renseigner les valeurs
python src/mailbot.py --dry-run     # montre ce qui serait fait : aucun envoi, aucun marquage
python src/mailbot.py --list        # liste les déclencheurs détectés
python src/mailbot.py               # traite les messages en attente
```

Dépendances : Python 3.11+ et la bibliothèque cliente Google (ou `imaplib`/`smtplib` pour un compte
IMAP classique). Les identifiants ne sont **jamais** dans le code : le fichier de jeton est indiqué par
`token_path` dans la configuration.

## Ce qui a été retiré avant publication

Ce dépôt a été **assaini** avant d'être publié. Retirés ou remplacés :

- les **adresses électroniques personnelles** réelles → `boite-principale@exemple.com` et
  `boite-assistant@exemple.com` ;
- les **chemins de jetons** propres à une machine → `token_path` dans la configuration ;
- l'**URL d'auto-hébergement** réelle → retirée ;
- tout fichier de **jeton, de session ou de secret** (aucun n'était présent dans le code, mais le
  `.gitignore` les exclut explicitement) ;
- les **configurations personnelles** (numéros, canaux, listes blanches) → vides dans l'exemple.

Le comportement n'a pas été modifié : ce sont des substitutions de valeurs, pas des coupes.

## Règles de sécurité

- **Jamais de jeton dans le dépôt.** Le `.gitignore` exclut `*.json` de configuration locale, les
  fichiers `token*`, `.env` et les journaux.
- **Le bot n'écrit qu'aux expéditeurs autorisés** et jamais aux adresses automatiques
  (`no-reply@…`, `noreply@…`, `donotreply@…`).
- **Mode à blanc par défaut dans le doute** : `--dry-run` n'envoie rien et ne marque rien.
- **Un message, un envoi.** L'état persistant est la seule protection contre la double réponse ; ne
  pas la contourner.
- Voir [SECURITY.md](SECURITY.md) pour signaler un problème.

## Licence

MIT — voir [LICENSE](LICENSE).
