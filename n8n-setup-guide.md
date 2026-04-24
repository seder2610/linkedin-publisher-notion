# n8n Setup Guide - LinkedIn Publisher Notion V1

## Vue d'ensemble

Trigger schedule -> lecture Notion (Ready + date <= maintenant) -> generation post -> update Notion -> notification.

---

## Nodes a creer (ordre)

1. `Schedule Trigger`
2. `Notion - Get many database pages`
3. `Edit Fields` (normaliser les champs)
4. `Basic LLM Chain`
5. `OpenAI Chat Model`
6. `Structured Output Parser`
7. `Code in JavaScript` (assembler content final propre)
8. `Notion - Update database page` (Status = Queued, Content Final)
9. `Notification node` (Slack/Email/Telegram)
10. `Notion - Update database page` (Status = Sent, Sent At)
11. `Error branch` -> update `Status = Error` + `Error Reason`

---

## 1) Schedule Trigger

- Tous les jours a heures fixes (ex: 08:30, 12:30, 18:30)
- Ou toutes les 30 min si tu veux plus reactif

---

## 2) Notion - Get many database pages

Resource: `Database Page`  
Operation: `Get many`  
Filtres:
- `Status` equals `Ready`
- `Channel` equals `LinkedIn`
- `Publish At` on or before `now`

Limit: 1 (au debut)

---

## 3) Edit Fields (normalisation)

Sortie conseillee:
- `page_id`
- `topic`
- `content_raw`
- `cta_type`
- `publish_at`

---

## 4) Basic LLM Chain + 5) OpenAI model + 6) Structured Parser

- Coller le prompt de `prompt-linkedin.md`
- Model: `gpt-4o-mini`
- Temperature: `0.6`

Schema parser:
```json
{
  "hook": "string",
  "body": "string",
  "cta": "string",
  "hashtags": ["string", "string", "string"],
  "content_final": "string"
}
```

---

## 7) Code node (assembler robuste)

Important: apres `Basic LLM Chain`, `$json` ne contient plus `page_id` / `topic` / etc.  
Il faut les recuperer depuis le node `Edit Fields` (ou fusionner les donnees avant).

Script recommande (remplace le nom du node si tu l as renomme):

```javascript
const out = $json.output || {};
const base = $item(0).$node["Edit Fields"].json || {};

// hashtags peut arriver en array OU en objet
let rawTags = [];
if (Array.isArray(out.hashtags)) {
  rawTags = out.hashtags;
} else if (out.hashtags && typeof out.hashtags === "object") {
  rawTags = Object.values(out.hashtags);
}

const hashtags = rawTags.filter(Boolean).map(h => String(h).trim());
const tagsLine = hashtags.length
  ? hashtags.map(h => h.startsWith('#') ? h : `#${h.replace(/\s+/g, '')}`).join(' ')
  : '';

const contentFinal = out.content_final && String(out.content_final).trim()
  ? String(out.content_final).trim()
  : [out.hook, out.body, out.cta, tagsLine].filter(Boolean).join('\n\n');

return [{
  json: {
    page_id: base.page_id || '',
    topic: base.topic || '',
    content_raw: base.content_raw || '',
    cta_type: base.cta_type || '',
    publish_at: base.publish_at || '',
    hook: out.hook || '',
    body: out.body || '',
    cta: out.cta || '',
    hashtags: tagsLine,
    content_final: contentFinal,
    sent_at: new Date().toISOString()
  }
}];
```

---

## 8) Notion Update (Queued + contenu)

Mise a jour page `page_id`:
- `Content Final` = `{{$json.content_final}}`
- `Status` = `Queued`
- `Error Reason` = vide
- `n8n Run ID` = `{{$execution.id}}`

---

## 9) Notification

Choisis un canal:
- Slack message
- Email
- Telegram

Message recommande:

```text
Post LinkedIn pret a publier:

{{$json.content_final}}

Page Notion: https://www.notion.so/{{$json.page_id}}
```

Note importante (Email SMTP):
- branche `Send Email` **apres** le node `Code` (sinon `$json.topic` / `$json.cta_type` / `$json.publish_at` seront vides si l entree vient d un node Notion ou Email).

---

## 10) Notion Update (Sent)

Ordre recommande:
`Code` -> `Notion Update (Queued)` -> `Send Email` -> `Notion Update (Sent)`

Dans `Notion Update (Sent)`, l entree immediate vient souvent de `Send Email` (qui ne contient pas `page_id`).  
Utilise des references explicites vers le node `Code`:

- `Database Page ID` = `{{ $node["Code in JavaScript"].json.page_id }}`
- `Status` = `Sent`
- `Sent At` = `{{ $node["Code in JavaScript"].json.sent_at }}`

Ne mappe pas `Sent At` / `Content Final` sur des proprietes Notion "id XX" au hasard: choisis les vrais champs `Sent At` / `Content Final` dans la liste.

---

## 11) Error handling (important)

Si une etape echoue:
- `Status` = `Error`
- `Error Reason` = message erreur

Tu peux utiliser `Continue On Fail` + branche IF sur erreur.

---

## Test rapide

1. Cree une ligne Notion:
   - Status: `Ready`
   - Channel: `LinkedIn`
   - Publish At: maintenant - 1 min
   - Content Raw: texte brut
2. Lance workflow
3. Verifie:
   - `Content Final` rempli
   - status passe `Queued` puis `Sent`
   - notification recue
