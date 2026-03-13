# MatchSense v3.0

**Outil d'aide à la décision pour le recrutement — Matching 3 CV / 1 Fiche de poste**

> 3 candidats, 1 poste — qui colle le mieux ?

MatchSense analyse 3 CV contre une fiche de poste en priorisant l'**engagement**, l'**alignement culturel** et les **soft skills**. Il détecte les **signaux faibles** et pénalise les **red flags relationnels**.

Les compétences techniques comptent, mais ne sont pas éliminatoires.

---

## Changement majeur en v3.0 : architecture multi-appels

### Le problème des versions précédentes

En v2.x, les 3 CV étaient envoyés dans **un seul appel** au modèle. Avec 36 000+ caractères de CV cumulés + un prompt système complexe, le modèle **confondait les candidats** : il attribuait les scores d'engagement d'un candidat à un autre, inventait de la "stabilité" pour un profil instable, et ne voyait pas le bénévolat d'un candidat engagé.

Ce n'est ni un problème de prompt ni de troncation — c'est un problème de **context tracking** sur des textes longs et structurellement similaires.

### La solution v3.0

```
AVANT (v2.x) :  1 appel → 3 CV ensemble → confusion
APRÈS (v3.0) :  3 appels parallèles → 1 CV chacun → assemblage client
```

Chaque appel ne voit qu'**un seul CV** contre la fiche de poste. Le modèle ne peut physiquement pas confondre les candidats. Les 3 appels partent en `Promise.all` (parallèle, pas de rallongement perceptible), puis le classement et la recommandation sont assemblés côté client en JavaScript.

### Avantages

- **Zéro confusion** entre candidats — chaque évaluation est isolée
- **Appels plus courts** (~20K tokens au lieu de ~50K) → modèle plus performant
- **Troncation portée à 15 000 chars/CV** puisqu'il n'y a plus de taille cumulative
- **Reproductibilité** améliorée — les scores ne dépendent plus de l'ordre des CV

---

## Multi-provider : OpenAI + Anthropic (Claude)

| Provider | Modèles | Clé API |
|----------|---------|---------|
| **OpenAI** | GPT-5.4, GPT-5.4 Pro, GPT-5.2, GPT-5.1, GPT-4o, GPT-4o Mini, GPT-4.1, GPT-4.1 Mini | `sk-proj-...` |
| **Anthropic** | Claude Sonnet 4 (recommandé), Claude Opus 4, Claude Haiku 4.5 | `sk-ant-api03-...` |

### Quel modèle choisir ?

- **Claude Sonnet 4** : recommandé — meilleur suivi des instructions structurées, moins de complaisance
- **GPT-5.4** : très performant, grand contexte, mais tendance à surnoter
- **Claude Opus 4** : max performance pour les cas complexes
- **GPT-4o / Claude Haiku 4.5** : rapides et économiques pour les tests

### Détails techniques

| | OpenAI | Anthropic |
|---|--------|-----------|
| Endpoint | `/v1/chat/completions` | `/v1/messages` |
| Auth | `Authorization: Bearer` | `x-api-key` + `anthropic-dangerous-direct-browser-access` |
| System prompt | Message `role: system` | Champ `system` dédié |
| JSON mode | `response_format: json_object` | Nettoyage backticks |

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
| 6-11 | 30-59% — lacunes significatives |
| 0-5 | Moins de 30% — métier différent |

**Règle d'or** : professionnel d'un autre métier sur un poste technique → 0-5/20 obligatoire.

### Pénalités red flags (soft skills)

| Red flag | Pénalité minimum |
|----------|-----------------|
| Remarque de manager sur la communication | -10 pts |
| Licenciement pour différend relationnel | -8 pts |
| CDD non renouvelé pour intégration | -6 pts |
| 10+ missions courtes sans prolongation | Signal fort |

### Plafond valeurs

Sans aucun signal d'engagement → maximum 15/40.

---

## Signaux faibles détectés

- Missions intérim/CDD dans l'entreprise recruteuse ou son secteur
- Métiers antérieurs avec contraintes similaires
- Polyvalence intersectorielle
- Progression rapide de responsabilités
- Engagement associatif, bénévolat, mentorat
- Auto-formation en parallèle d'un emploi
- Gestion de crise
- Reconversion professionnelle volontaire

---

## Conformité AI Act

| Exigence | Implémentation |
|----------|----------------|
| **Transparence** | Scores décomposés, justifications, pondération affichée |
| **Non-discrimination** | Anonymisation client-side (25+ patterns) + instruction modèle |
| **Contrôle humain** | "Score ≠ décision, l'humain reste décisionnaire" |
| **Minimisation** | Aucun stockage, clé effacée à la fermeture |
| **Auditabilité** | Prompt documenté, version taguée |

### Données anonymisées

Noms, civilités, emails, téléphones, adresses, codes postaux, dates de naissance, âge, nationalité, situation familiale, enfants, handicap/RQTH, n° sécu, IBAN, URLs, LinkedIn, photo.

---

## Sécurité (audit SGPT)

| # | Sévérité | Correctif |
|---|----------|-----------|
| 1 | CRITIQUE | XSS `file.name` → DOM pur (`textContent`) |
| 2 | CRITIQUE | Google Fonts → `crossorigin` + `referrerpolicy` |
| 3 | ÉLEVÉE | CSP → meta stricte, `connect-src` OpenAI + Anthropic |
| 4 | ÉLEVÉE | Clé API → base64, effacement `beforeunload`, validation format |
| 5 | ÉLEVÉE | En-têtes → X-Frame-Options, X-Content-Type-Options, Referrer-Policy |
| 6 | MOYENNE | Anonymisation → +10 patterns |
| 7 | MOYENNE | innerHTML → `esc()` sur tout contenu API |

---

## Utilisation

1. Ouvrir `matchsense-v3.0.html`
2. **Paramètres** → fournisseur (OpenAI/Anthropic) → clé API → modèle → **Enregistrer**
3. Déposer la **fiche de poste** (`.pdf`, `.txt`, `.docx`)
4. Renseigner les **valeurs et culture d'entreprise**
5. Déposer les **3 CV**
6. **C'est parti ?**

---

## Jeu de données de test

### CV n°1 — Sophie Le Guén (reconversion)

Ancienne boulangère, missions intérim SNCF, bénévolat Restos du Cœur.

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 2-4/20 | Métier différent, seul SAP basique |
| Valeurs | 25-32/40 | Bénévolat, tutorat apprentis, reconversion |
| Soft skills | 20-25/30 | Relation client, adaptation |
| Signaux faibles | 6-8/10 | Missions SNCF, contraintes similaires |
| **Global** | **~55-65** | |

### CV n°2 — Kévin Morel (technique instable)

14 postes en 12 ans, compétences techniques excellentes, red flags relationnels.

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 17-20/20 | Couvre quasi toutes les exigences |
| Valeurs | 5-12/40 | Plafonné — zéro engagement |
| Soft skills | 8-14/30 | Pénalités : licenciement, communication, instabilité |
| Signaux faibles | 1-3/10 | Peu de signaux positifs |
| **Global** | **~35-48** | |

### CV n°3 — Yassine Benmoussa (engagé stable)

CDI longs, formateur, tuteur, bénévole, trou de 2 ans. Nom arabe → teste l'anonymisation.

| Axe | Attendu | Raison |
|-----|---------|--------|
| Technique | 14-17/20 | Bonne couverture, un cran sous Kévin |
| Valeurs | 33-38/40 | Formateur, tuteur, délégué, bénévolat |
| Soft skills | 24-28/30 | Communication et mentorat prouvés |
| Signaux faibles | 7-9/10 | Stabilité, engagement associatif |
| **Global** | **~78-88** | **Classement #1 attendu** |

---

## Historique des versions

### v3.0 — Architecture multi-appels

**Problème** : le modèle confondait les candidats quand les 3 CV étaient dans le même appel. Kévin recevait les scores de Yassine et inversement.

**Fix** : 3 appels parallèles (1 CV par appel), assemblage côté client. Troncation à 15K chars/CV.

### v2.5 — Multi-provider

Ajout Anthropic (Claude Sonnet/Opus/Haiku), GPT-5.2, GPT-5.1.

### v2.4 — Troncation

**Problème** : CV tronqués à 4K chars, sections bénévolat coupées.

**Fix** : limites portées à 12K chars/CV.

### v2.3 — Anti-complaisance

**Problème** : GPT surnotait tout le monde.

**Fix** : analyse préalable obligatoire, seuils plancher, plafonds, température 0.05.

### v2.2 — Scoring relatif

**Problème** : compétences évaluées en absolu.

**Fix** : grille de calibration, scoring par rapport aux exigences du poste.

### v2.1-sec — Audit sécurité

7 vulnérabilités SGPT corrigées.

### v2.0 — 3 candidats

1 poste + 3 CV, zone valeurs, podium comparatif.

### v1.0 — Prototype

1 poste + 1 CV.

---

## Leçons de prompt engineering

### 1. "Sois honnête" ne suffit pas

Les LLMs ont un biais de complaisance. Seuls les **seuils plancher chiffrés** et les **exemples négatifs concrets** fonctionnent.

### 2. Forcer le raisonnement avant le scoring

Sans analyse préalable, le modèle score au feeling. Un champ obligatoire de pré-analyse force le chain-of-thought.

### 3. La troncation tue silencieusement

Les CV .docx extraits en texte brut font 10-17K chars. Tronquer à 4K coupe les sections engagement/bénévolat.

### 4. Les red flags doivent être des pénalités quantifiées

"Note les problèmes" → mention dans le commentaire mais pas d'impact sur le score. "Licenciement = -8 pts minimum" → impact réel.

### 5. Ne jamais mettre 3 CV dans le même appel

Le modèle confond les candidats sur des textes longs et similaires. **1 appel = 1 CV** est la seule architecture fiable.

### 6. La température compte

0.15 → 0.05 a réduit la variabilité significativement. Pour un outil de scoring, la reproductibilité prime.

### 7. Le choix du modèle change tout

Claude suit mieux les instructions structurées avec contraintes. GPT est plus créatif mais plus complaisant.

---

## Architecture

```
matchsense-v3.0.html
│
├── Paramètres multi-provider
│
├── anon()                    ← Anonymisation client-side (25+ regex)
│
├── 3× callAPI() en parallèle ← 1 CV par appel, Promise.all
│   ├── buildSystemPrompt()   ← Prompt individuel + seuils + pénalités
│   └── OpenAI ou Anthropic   ← Adapté au provider
│
├── Assemblage client          ← Tri par score, recommandation générée en JS
│
├── esc() + safeNum()          ← Sanitisation XSS
│
└── Rendu sécurisé             ← Podium + fiches + compliance
```

```
Fichier → extractText() → anon() → 3× API (parallèle) → assemblage → esc() → DOM
            client          client        réseau             client      client
```

---

## Limites connues

- **Extraction PDF basique** — préférer `.txt` ou `.docx`
- **Clé API côté client** — proxy backend nécessaire en production
- **Anonymisation regex** — couverture large mais pas infaillible
- **3 appels API** — coût ×3 par rapport à un appel unique (mais chaque appel est plus court)
- **Pas de comparaison inter-candidats côté modèle** — chaque évaluation est indépendante, les scores ne sont pas calibrés les uns par rapport aux autres

---

## Roadmap

- [ ] Fiche de poste de test (maintenance industrielle SNCF)
- [ ] 4ᵉ appel de synthèse comparative (envoyer les 3 scores au modèle pour une recommandation qualitative)
- [ ] Affichage de l'analyse préalable dans l'interface
- [ ] Proxy backend
- [ ] Extraction PDF via pdf.js
- [ ] Export résultats en PDF
- [ ] Mode 2 ou 5 candidats
- [ ] Comparaison automatique des résultats entre modèles

---

## Stack

- **Frontend** : HTML/CSS/JS vanilla, single-file, zéro dépendance
- **IA** : OpenAI (GPT-5.x / 4.x) + Anthropic (Claude) via `fetch`
- **Sécurité** : CSP, sanitisation XSS, anonymisation client-side
- **Prompt** : chain-of-thought forcé, seuils plancher, anti-complaisance, évaluation isolée

---

## Licence

Outil interne — usage restreint au périmètre de l'organisation.

---

*MatchSense v3.0 — 1 appel, 1 candidat, 0 confusion.*
