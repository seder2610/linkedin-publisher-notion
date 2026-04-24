# Notion Schema - LinkedIn Publisher V1

Creer une database Notion nommee: `LinkedIn Content Queue`

## Proprietes recommandees

- `Name` (Title)  
  Ex: "Post - AI automation recrutement"

- `Status` (Select)  
  Valeurs:
  - `Idea`
  - `Draft`
  - `Ready`
  - `Queued`
  - `Sent`
  - `Published`
  - `Error`

- `Topic` (Select)  
  Ex: Automation, IA, Odoo, Productivite, Recrutement

- `Content Raw` (Text)  
  Idee brute ou notes

- `Content Final` (Text)  
  Texte final genere/edite

- `Publish At` (Date)  
  Date/heure cible

- `Channel` (Select)  
  Valeurs:
  - `LinkedIn`

- `CTA Type` (Select)  
  Valeurs:
  - `Comment`
  - `DM`
  - `Follow`
  - `Save`

- `Hashtags` (Text)  
  Optionnel

- `Sent At` (Date)  
  Quand le message "pret a publier" est envoye

- `Published Url` (URL)  
  Rempli manuellement apres publication

- `Error Reason` (Text)  
  Message d'erreur si echec workflow

- `n8n Run ID` (Text)  
  Pour trace execution
