# MatchSense v2.5

**Outil d'aide à la décision pour le recrutement — Matching 3 CV / 1 Fiche de poste**

> 3 candidats, 1 poste — qui colle le mieux ?

MatchSense analyse 3 CV contre une fiche de poste en priorisant l'**engagement**, l'**alignement culturel** et les **soft skills**. Il détecte les **signaux faibles** que l'œil humain manque et pénalise les **red flags relationnels** que les outils classiques ignorent.

Les compétences techniques comptent, mais ne sont pas éliminatoires.

---

## Pourquoi cet outil existe

Les outils de matching classiques scorent les mots-clés techniques. Résultat : un technicien ultra-qualifié mais licencié 3 fois pour problèmes relationnels sort premier, et une personne en reconversion avec un parcours atypique mais engagée et alignée culturellement est écartée.

MatchSense inverse cette logique.

---

## Multi-provider : OpenAI + Anthropic (Claude)

La v2.5 supporte deux fournisseurs IA via un sélecteur dans les paramètres.

| Provider | Modèles disponibles | Clé API |
|----------|---------------------|---------|
| **OpenAI** | GPT-5.4 (recommandé), GPT-5.4 Pro, GPT-5.2, GPT-5.1, GPT-4o, GPT-4o Mini, GPT-4.1, GPT-4.1 Mini | `sk-proj-...` |
| **Anthropic** | Claude Sonnet 4 (recommandé), Claude Opus 4, Claude Haiku 4.5 | `sk-ant-api03-...` |

### Quel modèle choisir ?

- **Claude Sonnet 4** : recommandé pour le meilleur suivi des instructions structurées (analyse préalable, seuils plancher, pénalités). Plus rigoureux et moins sujet à la complaisance.
- **GPT-5.4** : très performant, grand contexte (1M tokens), bon suivi d'instructions mais tendance à surnoter.
- **Claude Opus 4** : max performance, idéal pour les cas complexes.
- **GPT-4o / Claude Haiku 4.5** : rapides et économiques pour les tests.

### Détails techniques d'intégration

| | OpenAI | Anthropic |
|---|--------|-----------|
| Endpoint | `/v1/chat/completions` | `/v1/messages` |
| Auth | `Authorization: Bearer sk-...` | `x-api-key: sk-ant-...` |
| System prompt | Message `role: system` | Champ `system` dédié |
| JSON mode | `response_format: { type: 'json_object' }` | Nettoyage des backticks ` ```json ``` ` |
| Header spécial | — | `anthropic-dangerous-direct-browser-access: true` |

---

## Pondération du scoring

| Axe | Points | Poids | Ce qui est évalué |
|-----|--------|-------|-------------------|
| **Valeurs & engagement** | /40 | 40% | Bénévolat, tutorat, stabilité, engagement collectif, projets transversaux |
| **Soft skills** | /30 | 30% | Communication, esprit d'équipe, fiabilité — avec pénalités pour les red flags |
| **Compétences techniques** | /20 | 20% | Adéquation STRICTE aux exigences du poste — non éliminatoire |
| **Signaux faibles** | /10 | 10% | Potentiel caché, reconversion, contraintes similaires |

---

## Comment le scoring fonctionne

### Étape 1 — Analyse préalable obligatoire

Avant de scorer, le modèle remplit un champ `analyse_prealable` contenant la liste des compétences techniques exigées par la fiche de poste, puis pour chaque candidat un mapping des compétences couvertes vs manquantes, des red flags et des signaux positifs. Cette étape force le raisonnement "faits d'abord, score ensuite".

### Étape 2 — Scoring technique par couverture

Le score technique mesure le pourcentage de compétences du poste couvertes par le candidat.

| Score | Couverture des exigences du poste |
|-------|-----------------------------------|
| 17-20 | 85%+ des compétences requises avec expérience prouvée |
| 12-16 | 60-84% des compétences requises |
| 6-11 | 30-59% — lacunes significatives |
| 0-5 | Moins de 30% — métier différent, compétences hors sujet |

**Règle d'or** : un professionnel d'un autre métier qui postule sur un poste technique obtient 0-5/20, même s'il est excellent dans son domaine.

### Étape 3 — Pénalités pour red flags

**Soft skills — pénalités :**

| Red flag | Pénalité minimum |
|----------|-----------------|
| Remarque explicite d'un manager sur la communication | -10 pts |
| Licenciement pour différend relationnel | -8 pts |
| CDD non renouvelé pour problèmes d'intégration | -6 pts |
| 10+ missions courtes sans jamais être prolongé | Signal fort de problème relationnel |

**Valeurs & engagement — plafond :**

Un candidat sans aucun signal d'engagement (zéro bénévolat, zéro tutorat, zéro projet collectif) est plafonné à 15/40 maximum.

### Étape 4 — Cohérence comparative

Les scores des 3 candidats doivent refléter les écarts réels entre leurs profils.

---

## Signaux faibles détectés

- Missions intérim/CDD réalisées dans l'entreprise recruteuse ou son secteur
- Métiers antérieurs avec contraintes similaires (nuit, astreintes, terrain, travail physique)
- Polyvalence démontrée par des changements de secteur réussis
- Progression rapide de responsabilités
- Engagement associatif, bénévolat, mentorat
- Auto-formation, certifications obtenues en parallèle d'un emploi
- Gestion de crise ou situations exceptionnelles
- Reconversion professionnelle volontaire

---

## Conformité AI Act

MatchSense est un **outil d'aide à la décision humaine** (AI Act, Article 14), pas un système de décision automatisée.

| Exigence | Implémentation |
|----------|----------------|
| **Transparence** | Score décomposé, justifications, analyse préalable, pondération affichée |
| **Non-discrimination** | Anonymisation côté client (25+ patterns) + instruction explicite au modèle |
| **Contrôle humain** | "Score ≠ décision, l'humain reste décisionnaire" |
| **Minimisation** | Aucune donnée stockée, clé API effacée à la fermeture |
| **Auditabilité** | Prompt documenté, version taguée, analyse préalable dans le JSON |

### Données anonymisées avant envoi

Noms/prénoms, civilités, emails, téléphones, adresses, codes postaux, dates de naissance, âge, nationalité, situation familiale, nombre d'enfants, handicap/RQTH/AAH/MDPH, numéro de sécurité sociale, IBAN, URLs, LinkedIn, réseaux sociaux, références photo.

---

## Sécurité (audit SGPT corrigé)

| # | Sévérité | Vulnérabilité | Correctif |
|---|----------|---------------|-----------|
| 1 | CRITIQUE | XSS via `innerHTML` avec `file.name` | DOM pur (`textContent`) |
| 2 | CRITIQUE | Google Fonts sans SRI | `crossorigin="anonymous"` + `referrerpolicy` |
| 3 | ÉLEVÉE | Absence de CSP | Meta CSP stricte, `connect-src` OpenAI + Anthropic |
| 4 | ÉLEVÉE | Clé API en mémoire | Encodage base64, effacement `beforeunload`, validation format |
| 5 | ÉLEVÉE | Absence d'en-têtes sécurité | X-Frame-Options, X-Content-Type-Options, Referrer-Policy |
| 6 | MOYENNE | Anonymisation insuffisante | +10 patterns (URLs, n° sécu, IBAN, noms, unicode) |
| 7 | MOYENNE | `innerHTML` non sanitisé | `esc()` sur tout contenu API, `safeNum()` pour les styles |

---

## Utilisation

### Prérequis

- Navigateur moderne (Chrome, Firefox, Edge, Safari)
- Clé API OpenAI ou Anthropic

### Lancement

1. Ouvrir `matchsense-v2.5.html`
2. **Paramètres** → choisir le fournisseur (OpenAI ou Anthropic) → saisir la clé API → choisir le modèle → **Enregistrer**
3. Déposer la **fiche de poste** (`.pdf`, `.txt`, `.docx`)
4. Renseigner les **valeurs et culture d'entreprise**
5. Déposer les **3 CV**
6. Cliquer sur **C'est parti ?**

---

## Jeu de données de test

Trois CV fictifs volontairement contrastés.

### CV n°1 — Sophie Le Guén (`CV_Sophie_LeGuen_Reconversion.docx`)

**Profil** : ancienne boulangère en reconversion, missions intérim SNCF

| Axe | Score attendu | Pourquoi |
|-----|---------------|----------|
| Technique | 2-4/20 | Métier différent, seul SAP basique comme point de contact |
| Valeurs | 25-32/40 | Bénévolat Restos du Cœur, tutorat apprentis CAP, reconversion volontaire |
| Soft skills | 20-25/30 | Relation client, esprit d'équipe, adaptation |
| Signaux faibles | 6-8/10 | Missions SNCF, contraintes similaires, reconversion |
| **Global** | **~55-65** | |

### CV n°2 — Kévin Morel (`CV2_Kevin_Morel_Maintenance.docx`)

**Profil** : ultra technique, instable, red flags relationnels

| Axe | Score attendu | Pourquoi |
|-----|---------------|----------|
| Technique | 17-20/20 | Couvre quasi toutes les exigences (automates, hydraulique, habilitations) |
| Valeurs | 5-12/40 | Plafonné — zéro bénévolat, zéro tutorat, zéro engagement |
| Soft skills | 8-14/30 | Pénalités : licenciement, "communication insuffisante", instabilité |
| Signaux faibles | 1-3/10 | Peu de signaux positifs |
| **Global** | **~35-48** | |

### CV n°3 — Yassine Benmoussa (`CV3_Yassine_Benmoussa.docx`)

**Profil** : bon technicien, stable, très engagé, trou de 2 ans

| Axe | Score attendu | Pourquoi |
|-----|---------------|----------|
| Technique | 14-17/20 | Bonne couverture (automates, GMAO, habilitations), un cran sous Kévin |
| Valeurs | 33-38/40 | Formateur, tuteur, délégué sécurité, bénévolat, coach foot |
| Soft skills | 24-28/30 | Communication, collaboration, mentorat prouvés |
| Signaux faibles | 7-9/10 | Stabilité, formation continue, engagement associatif |
| **Global** | **~78-88** | Classement #1 attendu |

Le nom à consonance arabe de Yassine teste que l'anonymisation neutralise tout biais.

---

## Historique des versions

### v2.5 — Multi-provider OpenAI + Anthropic

Ajout de Claude (Sonnet 4, Opus 4, Haiku 4.5) comme alternative à OpenAI. Sélecteur de fournisseur dans les paramètres, adaptation dynamique de la liste de modèles, gestion des deux formats d'API, CSP mise à jour. Ajout de GPT-5.2 et GPT-5.1 côté OpenAI.

### v2.4 — Correction de la troncation des CV

**Problème** : les CV étaient tronqués à 4 000 caractères. Le CV de Kévin (10 400 chars en .docx) n'était lu qu'à 38%, celui de Yassine (17 000 chars) à 23%. La section bénévolat/engagement de Yassine était coupée, et le pattern d'instabilité de Kévin (14 missions) n'était pas visible en entier.

**Fix** : troncation portée à 12 000 chars/CV et 8 000 chars pour la fiche de poste.

### v2.3 — Prompt anti-complaisance structurel

**Problème** : GPT surnotait tout le monde. Sophie (boulangère) obtenait 12/20 en technique. Kévin (instable) obtenait 30/40 en valeurs. Yassine arrivait dernier.

**Fix** : analyse préalable obligatoire, scoring par couverture, seuils plancher de pénalité, plafond valeurs, température 0.05.

### v2.2 — Scoring technique relatif

**Problème** : compétences évaluées en absolu au lieu d'être rapportées au poste.

**Fix** : grille de calibration, exemples concrets, instruction anti-complaisance.

### v2.1-sec — Audit de sécurité SGPT

7 vulnérabilités corrigées (2 critiques, 5 élevées, 2 moyennes).

### v2.0 — Passage à 3 candidats

1 fiche de poste + 3 CV, zone valeurs/culture, podium comparatif.

### v1.0 — Prototype initial

1 fiche de poste + 1 CV, scoring basique.

---

## Leçons de prompt engineering

Ce projet a traversé 5 itérations de prompt. Voici les enseignements clés.

### 1. "Sois honnête" ne suffit pas

Les LLMs ont un biais de complaisance (sycophancy). Dire "sois honnête" ou "utilise toute l'échelle" n'a quasiment aucun effet. Ce qui marche : des **seuils plancher chiffrés** ("un candidat sans engagement ≤ 15/40") et des **exemples négatifs concrets** ("boulanger sur poste maintenance = 0-5/20").

### 2. Forcer le raisonnement avant le scoring

Sans étape d'analyse préalable, le modèle score "au feeling" et produit des résultats incohérents. L'ajout d'un champ `analyse_prealable` obligatoire dans le JSON force un chain-of-thought structuré : lister les faits, mapper les compétences, identifier les red flags, PUIS scorer.

### 3. La troncation tue silencieusement

Tronquer les CV à 4 000 caractères semblait raisonnable — mais les CV .docx extraits en texte brut font facilement 10-17K caractères. Le modèle analysait des CV incomplets et inventait des qualités pour compenser. Toujours vérifier que le contenu arrive en entier.

### 4. Les red flags doivent être des pénalités, pas des observations

Dire au modèle de "noter les problèmes relationnels" produit une mention dans les commentaires mais pas d'impact sur le score. Il faut des **pénalités quantifiées** : "licenciement pour différend = -8 pts minimum en soft skills".

### 5. La température compte plus qu'on ne croit

Passer de 0.15 à 0.05 a réduit la variabilité inter-exécutions de façon significative. Pour un outil de scoring, la reproductibilité est plus importante que la créativité.

### 6. Le choix du modèle change tout

Claude est meilleur que GPT pour suivre des instructions structurées complexes avec des contraintes et des seuils. GPT est plus "créatif" dans ses justifications mais plus enclin à la complaisance.

---

## Architecture

```
matchsense-v2.5.html                ← Application single-file
│
├── Paramètres multi-provider        ← OpenAI ou Anthropic, modèles dynamiques
│
├── Anonymisation client-side        ← 25+ regex, AVANT tout envoi
│   └── anon() : noms, emails, tél, adresses, sécu, IBAN, LinkedIn...
│
├── Appel API (adapté au provider)   ← Seules les données anonymisées sortent
│   ├── System prompt structuré      ← Analyse préalable + seuils + pénalités
│   ├── Temperature: 0.05            ← Minimise la variance
│   └── JSON mode / nettoyage        ← response_format (OpenAI) ou regex (Claude)
│
├── Sanitisation des résultats       ← esc() + safeNum() sur tout contenu API
│
└── Rendu sécurisé                   ← Podium + fiches + compliance
```

### Flux de données

```
Fichiers → extractText() → anon() → API (OpenAI/Anthropic) → JSON → esc() → DOM
              client          client         réseau              client    client
```

Aucune donnée personnelle ne quitte le navigateur. Aucune donnée n'est stockée.

---

## Limites connues

- **Extraction PDF basique** : préférer `.txt` ou `.docx` pour de meilleurs résultats.
- **Clé API côté client** : acceptable en prototype, proxy backend nécessaire en production.
- **Anonymisation par regex** : couverture large mais pas infaillible.
- **Biais résiduel de complaisance** : malgré les garde-fous, vérifier que les écarts de scores reflètent les écarts réels. Claude est plus fiable que GPT sur ce point.
- **Appel Anthropic depuis le navigateur** : nécessite le header `anthropic-dangerous-direct-browser-access`. En production, passer par un backend.

---

## Roadmap

- [ ] Fiche de poste de test (maintenance industrielle SNCF)
- [ ] Affichage de l'analyse préalable dans l'interface (transparence)
- [ ] Proxy backend pour les clés API
- [ ] Extraction PDF via pdf.js
- [ ] Export résultats en PDF
- [ ] Mode 2 ou 5 candidats
- [ ] Tests A/B automatisés sur la stabilité du scoring
- [ ] Comparaison systématique des résultats entre modèles

---

## Stack

- **Frontend** : HTML/CSS/JS vanilla — single-file, zéro dépendance
- **IA** : OpenAI (GPT-5.x / 4.x) + Anthropic (Claude Sonnet/Opus/Haiku) via `fetch`
- **Sécurité** : CSP, sanitisation XSS, anonymisation client-side, en-têtes HTTP
- **Prompt engineering** : chain-of-thought forcé, analyse préalable, seuils plancher, anti-complaisance

---

## Licence

Outil interne — usage restreint au périmètre de l'organisation.

---

*MatchSense v2.5 — Conçu pour trouver le signal dans le bruit, pas pour faire plaisir.*
