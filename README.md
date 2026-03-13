# MatchSense v2.3

**Outil d'aide à la décision pour le recrutement — Matching 3 CV / 1 Fiche de poste**

> 3 candidats, 1 poste — qui colle le mieux ?

MatchSense analyse 3 CV contre une fiche de poste en priorisant l'**engagement**, l'**alignement culturel** et les **soft skills**. Il détecte les **signaux faibles** que l'œil humain manque et pénalise les **red flags relationnels** que les recruteurs voient mais que les outils classiques ignorent.

Les compétences techniques comptent, mais ne sont pas éliminatoires.

---

## Pourquoi cet outil existe

Les outils de matching classiques scorent les mots-clés techniques. Résultat : un technicien ultra-qualifié mais licencié 3 fois pour problèmes relationnels sort premier, et une personne en reconversion avec un parcours atypique mais engagée et alignée culturellement est écartée.

MatchSense inverse cette logique.

---

## Pondération du scoring

| Axe | Points | Poids | Ce qui est évalué |
|-----|--------|-------|-------------------|
| **Valeurs & engagement** | /40 | 40% | Bénévolat, tutorat, stabilité, engagement collectif, projets transversaux |
| **Soft skills** | /30 | 30% | Communication, esprit d'équipe, fiabilité — avec pénalités pour les red flags |
| **Compétences techniques** | /20 | 20% | Adéquation STRICTE aux exigences du poste — non éliminatoire |
| **Signaux faibles** | /10 | 10% | Potentiel caché, reconversion, contraintes similaires |

---

## Comment le scoring fonctionne (v2.3)

### Étape 1 — Analyse préalable obligatoire

Avant de scorer, le modèle doit remplir un champ `analyse_prealable` dans le JSON qui contient la liste des compétences techniques exigées par la fiche de poste, puis pour chaque candidat un mapping explicite des compétences couvertes vs manquantes, des red flags identifiés et des signaux positifs détectés. Cette étape force le raisonnement "faits d'abord, score ensuite" et empêche le modèle de scorer à l'intuition.

### Étape 2 — Scoring technique par couverture

Le score technique n'est plus évalué "au feeling". Le modèle doit lister les compétences du poste, cocher celles que le candidat possède, barrer celles qu'il n'a pas, et scorer en proportion.

| Score | Couverture des exigences du poste |
|-------|-----------------------------------|
| 17-20 | 85%+ des compétences requises avec expérience prouvée |
| 12-16 | 60-84% des compétences requises |
| 6-11 | 30-59% — lacunes significatives |
| 0-5 | Moins de 30% — métier différent, compétences hors sujet |

**Règle d'or** : un professionnel d'un autre métier (boulanger, commercial, agent de nettoyage) qui postule sur un poste technique (maintenance, informatique, ingénierie) obtient obligatoirement entre 0 et 5, même s'il est excellent dans son domaine.

### Étape 3 — Pénalités pour red flags

Les red flags relationnels et comportementaux sont pénalisés avec des seuils plancher explicites.

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

Les scores des 3 candidats doivent refléter les écarts réels. Si le candidat B maîtrise 3 automates Siemens et que le candidat A n'a jamais touché un automate, l'écart de score technique doit être massif — pas 2 points d'écart.

---

## Signaux faibles détectés

- Missions intérim/CDD réalisées dans l'entreprise recruteuse ou son secteur
- Métiers antérieurs avec contraintes similaires (nuit, astreintes, terrain, travail physique)
- Polyvalence démontrée par des changements de secteur réussis
- Progression rapide de responsabilités
- Engagement associatif, bénévolat, mentorat
- Auto-formation, certifications obtenues en parallèle d'un emploi
- Gestion de crise ou situations exceptionnelles
- Implication dans des projets transversaux ou d'innovation
- Reconversion professionnelle volontaire

---

## Conformité AI Act

MatchSense est conçu comme un **outil d'aide à la décision humaine** (AI Act, Article 14), pas comme un système de décision automatisée.

| Exigence | Implémentation |
|----------|----------------|
| **Transparence** | Score décomposé par axe, justifications textuelles, analyse préalable visible, pondération affichée |
| **Non-discrimination** | Anonymisation côté client (25+ patterns) + instruction explicite au modèle d'ignorer tout critère protégé |
| **Contrôle humain** | Mention systématique "score ≠ décision, l'humain reste décisionnaire" |
| **Minimisation** | Aucune donnée stockée, clé API effacée à la fermeture |
| **Auditabilité** | Prompt documenté, version taguée, analyse préalable dans le JSON |

### Données anonymisées avant envoi

L'anonymisation côté client supprime : noms/prénoms, civilités, emails, téléphones, adresses, codes postaux, dates de naissance, âge, nationalité, situation familiale, nombre d'enfants, mentions de handicap/RQTH/AAH/MDPH, numéro de sécurité sociale, IBAN, URLs, profils LinkedIn et réseaux sociaux, références à une photo.

---

## Sécurité (audit SGPT corrigé)

| # | Sévérité | Vulnérabilité | Correctif |
|---|----------|---------------|-----------|
| 1 | CRITIQUE | XSS via `innerHTML` avec `file.name` | `renderFileLoaded` réécrit en DOM pur (`textContent`) |
| 2 | CRITIQUE | Google Fonts sans SRI | `crossorigin="anonymous"` + `referrerpolicy="no-referrer"` |
| 3 | ÉLEVÉE | Absence de CSP | Meta CSP stricte, `connect-src` limité à `api.openai.com`, `frame-ancestors 'none'` |
| 4 | ÉLEVÉE | Clé API en mémoire | Encodage base64, getter/setter, effacement au `beforeunload`, validation `sk-` |
| 5 | ÉLEVÉE | Absence d'en-têtes sécurité | `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy` |
| 6 | MOYENNE | Anonymisation insuffisante | +10 patterns (URLs, n° sécu, IBAN, noms propres, unicode étendu) |
| 7 | MOYENNE | `innerHTML` non sanitisé | Fonction `esc()` sur tout contenu API, `safeNum()` pour les styles |

---

## Utilisation

### Prérequis

- Navigateur moderne (Chrome, Firefox, Edge, Safari)
- Clé API OpenAI avec accès GPT-5.4 ou GPT-4o

### Lancement

1. Ouvrir `matchsense-v2.3.html` dans un navigateur
2. **Paramètres** → saisir la clé API OpenAI → **Enregistrer**
3. Déposer la **fiche de poste** (drag & drop ou clic)
4. Renseigner les **valeurs et culture d'entreprise**
5. Déposer les **3 CV** des candidats A, B et C
6. Cliquer sur **C'est parti ?**

### Formats acceptés

`.pdf`, `.txt`, `.docx` — le `.txt` donne les meilleurs résultats, le `.docx` fonctionne bien, le `.pdf` utilise une extraction basique.

### Résultats

- **Podium comparatif** avec classement, scores globaux et barres par axe
- **Recommandation comparative** en prose
- **Fiches détaillées** par candidat : justification de chaque axe, signaux faibles, points forts/vigilance, synthèse
- **Disclaimer AI Act**

---

## Jeu de données de test

Trois CV fictifs couvrent des profils volontairement contrastés pour tester les garde-fous du scoring.

### CV n°1 — Sophie Le Guén (`CV_Sophie_LeGuen_Reconversion.docx`)

**Profil** : ancienne boulangère en reconversion, missions intérim à la SNCF

- 10 ans d'artisanat boulanger, puis reconversion volontaire
- 2 missions intérim au Technicentre de Maintenance SNCF Nantes (nettoyage puis logistique)
- Bénévolat aux Restos du Cœur, formatrice d'apprentis CAP, relation client
- Compétences techniques hors sujet pour un poste maintenance industrielle

**Scores attendus** : technique 2-4/20 (SAP basique = seul point de contact) · valeurs/engagement 25-32/40 (bénévolat, tutorat apprentis, reconversion) · soft skills 20-25/30 · signaux faibles 6-8/10 (missions SNCF, contraintes similaires) · **global ~55-65**

### CV n°2 — Kévin Morel (`CV2_Kevin_Morel_Maintenance.docx`)

**Profil** : ultra technique, instable, problèmes relationnels documentés

- 14 postes en 12 ans, majorité en intérim 2-6 mois
- Compétences techniques excellentes (automates Siemens/Schneider/Allen-Bradley, habilitations, GMAO)
- Red flags explicites : "difficultés d'intégration", "communication insuffisante", licenciement pour différend, démissions multiples
- Zéro engagement, zéro bénévolat, centres d'intérêt solitaires

**Scores attendus** : technique 17-20/20 · valeurs/engagement 5-12/40 (plafonné, zéro signal) · soft skills 8-14/30 (pénalités red flags) · signaux faibles 1-3/10 · **global ~35-48**

### CV n°3 — Yassine Benmoussa (`CV3_Yassine_Benmoussa.docx`)

**Profil** : bon technicien, stable, très engagé, trou de 2 ans dans le CV

- 4 CDI longs (jusqu'à 4 ans 6 mois chez Airbus Atlantic)
- Trou de 2 ans (2019-2021) pour raisons personnelles
- Signaux forts : formateur sécurité, tuteur 2 alternants, référent formation 12 opérateurs, participant TPM, délégué sécurité
- Bénévolat : soutien scolaire, coach foot enfants, courses caritatives
- Nom à consonance arabe → teste que l'anonymisation neutralise tout biais

**Scores attendus** : technique 14-17/20 · valeurs/engagement 33-38/40 · soft skills 24-28/30 · signaux faibles 7-9/10 · **global ~78-88** — classement #1 attendu

---

## Historique des versions

### v2.3 — Prompt anti-complaisance structurel

**Problème** : GPT surnotait tous les candidats par complaisance. Sophie (boulangère) obtenait 12/20 en technique sur un poste maintenance. Kévin (instable, 0 engagement) obtenait 30/40 en valeurs. Yassine (le meilleur profil) arrivait dernier.

**Cause racine** : le prompt demandait un scoring global sans forcer de raisonnement préalable. Le modèle scorait "au feeling" sans comparer les candidats ni lister les compétences du poste.

**Correctifs** : analyse préalable obligatoire, scoring par couverture, seuils plancher de pénalité, plafond valeurs si zéro engagement, cohérence comparative, température 0.05.

### v2.2 — Scoring technique relatif

**Problème** : le modèle évaluait les compétences techniques de manière absolue au lieu de les rapporter aux exigences du poste.

**Correctifs** : grille de calibration, exemples concrets, règle anti-complaisance.

### v2.1-sec — Audit de sécurité SGPT

7 vulnérabilités corrigées (2 critiques, 5 élevées, 2 moyennes).

### v2.0 — Passage à 3 candidats

Refonte complète : 1 fiche de poste + 3 CV, zone valeurs/culture, podium comparatif, pondération engagement-first.

### v1.0 — Prototype initial

1 fiche de poste + 1 CV, scoring basique.

---

## Architecture

```
matchsense-v2.3.html              ← Application single-file
│
├── Anonymisation client-side      ← 25+ regex, AVANT tout envoi
│   └── anon() : emails, tél, adresses, noms, sécu, IBAN, LinkedIn...
│
├── Appel API OpenAI               ← Seules les données anonymisées sortent
│   ├── System prompt structuré    ← Analyse préalable + règles strictes
│   ├── Temperature: 0.05          ← Minimise la variance
│   └── JSON mode                  ← response_format: json_object
│
├── Sanitisation des résultats     ← esc() + safeNum() sur tout contenu API
│
└── Rendu sécurisé                 ← Podium + fiches détaillées + compliance
```

### Flux de données

```
Fichiers → extractText() → anon() → API OpenAI → JSON → esc() → DOM
              client          client      réseau      client    client
```

Aucune donnée personnelle ne quitte le navigateur. Aucune donnée n'est stockée.

---

## Modèles supportés

| Modèle | Usage | Notes |
|--------|-------|-------|
| `gpt-5.4` | Recommandé | Meilleur suivi des instructions structurées |
| `gpt-5.4-pro` | Max performance | Responses API uniquement |
| `gpt-4o` | Fallback fiable | Bon compromis qualité/coût |
| `gpt-4o-mini` | Tests rapides | Moins fiable sur le suivi des règles strictes |
| `gpt-4.1` / `gpt-4.1-mini` | Alternatives | À tester |

---

## Limites connues

- **Extraction PDF basique** : les PDF complexes peuvent perdre du contenu. Préférer `.txt` ou `.docx`.
- **Clé API côté client** : acceptable en prototype, proxy backend nécessaire en production.
- **Anonymisation par regex** : couverture large mais pas infaillible.
- **Biais résiduel de complaisance** : malgré les garde-fous, certains modèles peuvent encore surnoter. Vérifier les écarts.

---

## Roadmap

- [ ] Fiche de poste de test (maintenance industrielle SNCF)
- [ ] Affichage de l'analyse préalable dans l'interface
- [ ] Proxy backend pour la clé API
- [ ] Extraction PDF via pdf.js
- [ ] Export résultats en PDF
- [ ] Mode 2 ou 5 candidats
- [ ] Tests A/B automatisés sur la stabilité du scoring

---

## Stack

- **Frontend** : HTML/CSS/JS vanilla — single-file, zéro dépendance
- **IA** : API OpenAI (GPT-5.4 / GPT-4o) via `fetch`
- **Sécurité** : CSP, sanitisation XSS, anonymisation client-side, en-têtes HTTP
- **Prompt engineering** : chain-of-thought forcé, analyse préalable, seuils plancher, anti-complaisance

---

## Licence

Outil interne — usage restreint au périmètre de l'organisation.

---

*MatchSense v2.3 — Conçu pour trouver le signal dans le bruit, pas pour faire plaisir.*
