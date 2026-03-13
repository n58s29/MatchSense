# MatchSense v3.1

**Outil d'aide à la décision pour le recrutement — Matching 3 CV / 1 Fiche de poste**

> 3 candidats, 1 poste — qui colle le mieux ?

MatchSense analyse 3 CV contre une fiche de poste en priorisant l'**engagement**, l'**alignement culturel** et les **soft skills**. Il détecte les **signaux faibles** et pénalise les **red flags relationnels**.

Les compétences techniques comptent, mais ne sont pas éliminatoires.

---

## Comment ça marche

1. Déposer 1 fiche de poste + décrire les valeurs de l'entreprise
2. Déposer 3 CV (`.docx`, `.txt`, `.pdf`)
3. L'outil lance **3 appels API en parallèle** (1 CV par appel, pas de confusion)
4. Chaque candidat est scoré individuellement puis classé côté client

---

## Architecture multi-appels (v3.0+)

```
AVANT (v2.x) :  1 appel → 3 CV ensemble → le modèle confond les candidats
APRÈS (v3.0+) :  3 appels parallèles → 1 CV chacun → assemblage client
```

Chaque appel ne voit qu'**un seul CV** contre la fiche de poste. `Promise.all` lance les 3 en parallèle. Le classement est calculé côté client.

---

## Extraction DOCX native (v3.1)

### Le bug critique des versions précédentes

Un `.docx` est une **archive ZIP** contenant du XML. L'ancien extracteur faisait `file.text()` sur le binaire et essayait de récupérer les caractères ASCII imprimables. Résultat : **bouillie illisible**. Le modèle recevait du charabia et répondait "CV illisible, aucune compétence identifiable" — ce qui expliquait les scores à 0/0/0 pour tous les candidats.

### Le fix v3.1

Un vrai parseur DOCX natif en 3 étapes, **zéro dépendance externe** :

1. **Parcourir les entrées ZIP** — lecture des Local File Headers (`PK\x03\x04`), extraction des noms et positions
2. **Décompresser** `word/document.xml` — via `DecompressionStream('deflate-raw')` (API native navigateur)
3. **Parser le XML** — fins de paragraphes `</w:p>` → retours à la ligne, suppression des balises, décodage des entités XML (`&amp;`, `&#x2019;`, etc.)

### Résultats de test sur les 3 CV

| CV | Chars extraits | Bénévolat | Tutorat | Licenciement | Problèmes intégration |
|----|---------------|-----------|---------|--------------|----------------------|
| Sophie Le Guén | 3 463 | ✅ | ✅ | — | — |
| Kévin Morel | 4 766 | — | — | ✅ | ✅ |
| Yassine Benmoussa | 3 537 | ✅ | ✅ | — | — |

Tous les signaux clés sont correctement extraits — plus de CV "illisible".

---

## Multi-provider : OpenAI + Anthropic

| Provider | Modèles | Clé API |
|----------|---------|---------|
| **OpenAI** | GPT-5.4, GPT-5.4 Pro, GPT-5.2, GPT-5.1, GPT-4o, GPT-4o Mini, GPT-4.1, GPT-4.1 Mini | `sk-proj-...` |
| **Anthropic** | Claude Sonnet 4 (recommandé), Claude Opus 4, Claude Haiku 4.5 | `sk-ant-api03-...` |

### Quel modèle choisir ?

- **Claude Sonnet 4** : recommandé — meilleur suivi des instructions structurées, moins de complaisance
- **GPT-5.4** : performant, grand contexte, mais tendance à surnoter
- **Claude Opus 4** : max performance pour les cas complexes
- **GPT-4o / Haiku 4.5** : rapides et économiques pour les tests

---

## Pondération du scoring

| Axe | Points | Poids | Ce qui est évalué |
|-----|--------|-------|-------------------|
| **Valeurs & engagement** | /40 | 40% | Bénévolat, tutorat, stabilité, engagement collectif |
| **Soft skills** | /30 | 30% | Communication, esprit d'équipe — pénalités pour red flags |
| **Compétences techniques** | /20 | 20% | Adéquation STRICTE aux exigences du poste |
| **Signaux faibles** | /10 | 10% | Potentiel caché, reconversion, contraintes similaires |

---

## Mécanismes anti-complaisance

### Scoring technique par couverture

| Score | Couverture des exigences du poste |
|-------|-----------------------------------|
| 17-20 | 85%+ avec expérience prouvée |
| 12-16 | 60-84% |
| 6-11 | 30-59% |
| 0-5 | Moins de 30% — métier différent |

**Règle d'or** : professionnel d'un autre métier sur un poste technique → 0-5/20.

### Pénalités red flags

| Red flag | Pénalité |
|----------|---------|
| Remarque manager sur la communication | -10 pts |
| Licenciement pour différend relationnel | -8 pts |
| CDD non renouvelé pour intégration | -6 pts |
| 10+ missions courtes sans prolongation | Signal fort |

### Plafond valeurs

Sans signal d'engagement → maximum 15/40.

---

## Conformité AI Act

| Exigence | Implémentation |
|----------|----------------|
| **Transparence** | Scores décomposés, justifications, pondération affichée |
| **Non-discrimination** | Anonymisation client-side (25+ patterns) |
| **Contrôle humain** | "Score ≠ décision" |
| **Minimisation** | Aucun stockage, clé effacée à la fermeture |

### Données anonymisées

Noms, civilités, emails, téléphones, adresses, codes postaux, dates de naissance, âge, nationalité, situation familiale, enfants, handicap/RQTH, n° sécu, IBAN, URLs, LinkedIn, photo.

---

## Sécurité (audit SGPT)

| # | Sévérité | Correctif |
|---|----------|-----------|
| 1 | CRITIQUE | XSS `file.name` → DOM pur |
| 2 | CRITIQUE | Google Fonts → `crossorigin` + `referrerpolicy` |
| 3 | ÉLEVÉE | CSP stricte, `connect-src` OpenAI + Anthropic |
| 4 | ÉLEVÉE | Clé API → base64, effacement `beforeunload` |
| 5 | ÉLEVÉE | En-têtes sécurité HTTP |
| 6 | MOYENNE | Anonymisation +10 patterns |
| 7 | MOYENNE | `esc()` sur tout contenu API |

---

## Jeu de données de test

### CV n°1 — Sophie Le Guén (reconversion)

Ancienne boulangère, missions intérim SNCF, bénévolat Restos du Cœur.

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 2-4/20 | Métier différent |
| Valeurs | 25-32/40 | Bénévolat, tutorat apprentis |
| Soft skills | 20-25/30 | Relation client, adaptation |
| Signaux faibles | 6-8/10 | Missions SNCF, reconversion |
| **Global** | **~55-65** | |

### CV n°2 — Kévin Morel (technique instable)

14 postes en 12 ans, compétences excellentes, red flags relationnels.

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 17-20/20 | Couvre quasi toutes les exigences |
| Valeurs | 5-12/40 | Plafonné — zéro engagement |
| Soft skills | 8-14/30 | Licenciement, communication, instabilité |
| Signaux faibles | 1-3/10 | Peu de signaux positifs |
| **Global** | **~35-48** | |

### CV n°3 — Yassine Benmoussa (engagé stable)

CDI longs, formateur, tuteur, bénévole, trou de 2 ans. Teste l'anonymisation (nom arabe).

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 14-17/20 | Bonne couverture |
| Valeurs | 33-38/40 | Formateur, tuteur, bénévolat |
| Soft skills | 24-28/30 | Communication et mentorat |
| Signaux faibles | 7-9/10 | Stabilité, engagement |
| **Global** | **~78-88** | **#1 attendu** |

---

## Historique des versions

### v3.1 — Extraction DOCX native

**Problème** : l'extracteur .docx ne fonctionnait pas — il lisait le binaire ZIP brut au lieu de décompresser le XML. Les CV arrivaient illisibles au modèle, qui scorait tout à 0.

**Fix** : parseur ZIP natif + `DecompressionStream('deflate-raw')` + extraction du texte depuis `word/document.xml`. Zéro dépendance.

### v3.0 — Architecture multi-appels

**Problème** : avec 3 CV dans le même appel, le modèle confondait les candidats (attribuait les scores de l'un à l'autre).

**Fix** : 3 appels parallèles (1 CV par appel), assemblage client.

### v2.5 — Multi-provider

Ajout Anthropic (Claude), GPT-5.2, GPT-5.1.

### v2.4 — Troncation

CV tronqués à 4K chars → sections coupées. Fix : 12K puis 15K chars.

### v2.3 — Anti-complaisance

GPT surnotait tout. Fix : analyse préalable, seuils plancher, température 0.05.

### v2.2 — Scoring relatif

Compétences évaluées en absolu. Fix : scoring par rapport au poste.

### v2.1-sec — Audit sécurité

7 vulnérabilités SGPT corrigées.

### v2.0 — 3 candidats

1 poste + 3 CV, zone valeurs, podium.

### v1.0 — Prototype

1 poste + 1 CV.

---

## Leçons de prompt engineering

| # | Leçon | Détail |
|---|-------|--------|
| 1 | "Sois honnête" ne suffit pas | Seuls les seuils plancher chiffrés et les exemples négatifs fonctionnent |
| 2 | Forcer le raisonnement avant le scoring | Champ `analyse` obligatoire dans le JSON = chain-of-thought forcé |
| 3 | La troncation tue silencieusement | Les .docx font 10-17K chars en texte brut — tronquer à 4K coupe les sections clés |
| 4 | Les red flags doivent être des pénalités quantifiées | "Licenciement = -8 pts" fonctionne, "note les problèmes" ne fonctionne pas |
| 5 | Ne jamais mettre 3 CV dans le même appel | Le modèle confond les candidats — 1 appel = 1 CV |
| 6 | La température compte | 0.15 → 0.05 = variabilité réduite significativement |
| 7 | Le choix du modèle change tout | Claude suit mieux les instructions structurées, GPT est plus complaisant |
| 8 | Vérifier que les données arrivent | Le bug le plus vicieux : le modèle reçoit de la bouillie et invente au lieu de dire "illisible" |

---

## Architecture

```
matchsense-v3.1.html
│
├── extractDocx()              ← Parseur ZIP natif + DecompressionStream
│
├── anon()                     ← Anonymisation client-side (25+ regex)
│
├── 3× callAPI() parallèles   ← 1 CV par appel, Promise.all
│   ├── buildSystemPrompt()    ← Prompt individuel + seuils + pénalités
│   └── OpenAI ou Anthropic    ← Adapté au provider
│
├── Assemblage client          ← Tri par score, recommandation JS
│
├── esc() + safeNum()          ← Sanitisation XSS
│
└── Rendu sécurisé             ← Podium + fiches + compliance
```

```
.docx → extractDocx() → anon() → callAPI() → JSON → esc() → DOM
  ZIP      XML→texte      regex     réseau      client    client
```

---

## Limites connues

- **Extraction PDF basique** — préférer `.docx` ou `.txt`
- **Clé API côté client** — proxy backend nécessaire en production
- **3 appels API** — coût ×3 mais chaque appel est plus court et plus fiable
- **Pas de comparaison inter-candidats côté modèle** — les scores sont indépendants

---

## Roadmap

- [ ] 4ᵉ appel de synthèse comparative (recommandation qualitative par le modèle)
- [ ] Fiche de poste de test
- [ ] Affichage de l'analyse dans l'interface
- [ ] Proxy backend
- [ ] Export résultats en PDF
- [ ] Mode 2 ou 5 candidats
- [ ] Extraction PDF via pdf.js

---

## Stack

- **Frontend** : HTML/CSS/JS vanilla, single-file, zéro dépendance
- **Extraction** : parseur ZIP/DOCX natif via `DecompressionStream`
- **IA** : OpenAI (GPT-5.x / 4.x) + Anthropic (Claude) via `fetch`
- **Sécurité** : CSP, sanitisation XSS, anonymisation client-side
- **Prompt** : chain-of-thought forcé, seuils plancher, évaluation isolée

---

## Licence

Outil interne — usage restreint au périmètre de l'organisation.

---

*MatchSense v3.1 — 1 appel, 1 candidat, 1 vrai extracteur DOCX.*
