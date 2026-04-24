# LinkedIn Publisher Notion V1 (n8n)

Automatisation V1 pour publier (ou preparer publication) sur LinkedIn a partir d'une database Notion.

## Objectif

Passer d'idees brutes dans Notion a des posts LinkedIn prets a publier, avec planning et suivi.

## Principe V1 (safe)

LinkedIn est parfois limite pour les profils personnels via API.  
La V1 privilegie une approche fiable:

1. Detecter les contenus `Ready` dans Notion
2. Generer/ameliorer le post (hook + body + CTA + hashtags)
3. Planifier selon `publish_at`
4. Envoyer une notification "Pret a publier" avec texte final
5. Marquer dans Notion (`Queued` -> `Published` manuel ou `Sent`)

## Pourquoi cette V1 est utile

- Zero friction de production
- Moins de risque de blocage API LinkedIn
- Tu gardes le controle final avant publication
- Tu peux evoluer vers V2 auto-post plus tard

## Fichiers

- `notion-schema.md` : structure de la database Notion
- `n8n-setup-guide.md` : construction node par node
- `prompt-linkedin.md` : prompt de generation/optimisation post

## Definition of done

- 1 item Notion `Ready` detecte
- post final genere automatiquement
- notification recue (Slack/Email/Telegram)
- statut Notion mis a jour correctement

## Export GitHub (ce que tu pushes)

Dans n8n :
1. Ouvre le workflow `linkedin-publisher-v1`
2. Menu `...` -> `Download` (export JSON)
3. Renomme le fichier en `linkedin-publisher-notion-v1.json`
4. Place-le dans ce dossier : `Output/linkedin-publisher-notion-v1/`
5. Commit + push sur ton repo GitHub (nouveau repo dédié recommandé)

Note : je ne peux pas generer un JSON fidele sans ton export n8n (IDs de nodes, versions). Le fichier exporte est la source de verite.

## Roadmap (ne pas perdre l objectif final)

**Objectif final** : ne plus aller sur LinkedIn pour publier.

- **V1 (actuelle)** : generation + file Notion + notification (fiabilite maximale)
- **V2 (auto-post)** : publication via **LinkedIn API** (souvent necessite une **Organization Page** / produits LinkedIn + droits) OU via un connecteur officiel si disponible sur ton compte

Regle simple : tant que tu postes en **profil personnel**, l API LinkedIn est souvent plus contrainte qu en **page entreprise**. Si ton objectif est 100% sans UI LinkedIn, on choisit volontairement le canal le plus compatible API (souvent Page) puis on branche un node `HTTP Request` / node LinkedIn selon ce qui est dispo sur ton stack n8n.
