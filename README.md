# MatchSense v2.2-sec

**Outil d'aide à la décision pour le recrutement — Matching CV / Fiche de poste**

> 3 candidats, 1 poste — qui colle le mieux ?

MatchSense analyse 3 CV contre une fiche de poste en priorisant l'**engagement**, l'**alignement culturel** et les **soft skills** plutôt que les seules compétences techniques. Il détecte les **signaux faibles** que l'œil humain manque souvent.

---

## Philosophie

Les recruteurs écartent trop vite des profils atypiques : parcours non linéaires, trous dans le CV, reconversions, missions d'intérim multiples. MatchSense inverse la logique habituelle :

- **Les compétences techniques ne sont pas éliminatoires.** Un candidat moins technique mais très engagé peut scorer plus haut qu'un expert isolé.
- **Les signaux faibles sont valorisés.** Bénévolat, tutorat, engagement associatif, progression rapide, polyvalence intersectorielle — tout ce qui ne rentre pas dans une grille classique.
- **Zéro biais.** Anonymisation côté client avant tout envoi. Le modèle IA reçoit l'instruction explicite d'ignorer tout indice de genre, âge, origine, handicap, situation familiale.

---

## Pondération du scoring

| Axe | Points | Poids | Philosophie |
|-----|--------|-------|-------------|
| **Valeurs & engagement** | /40 | 40% | Le plus important — alignement avec la culture d'entreprise |
| **Soft skills** | /30 | 30% | Savoir-être, communication, esprit d'équipe, adaptabilité |
| **Compétences techniques** | /20 | 20% | Évaluées mais **non éliminatoires** |
| **Signaux faibles** | /10 | 10% | Bonus — détection de potentiel caché |

### Signaux faibles détectés

- Missions intérim/CDD dans le même secteur ou la même entreprise
- Métiers antérieurs avec contraintes similaires (nuit, astreintes, terrain)
- Polyvalence démontrée par des changements de secteur réussis
- Progression rapide de responsabilités
- Engagement associatif, bénévolat, mentorat
- Auto-formation, certifications obtenues en parallèle d'un emploi
- Gestion de crise ou situations exceptionnelles
- Implication dans des projets transversaux ou d'innovation

---

## Conformité AI Act

MatchSense est conçu comme un **outil d'aide à la décision humaine** (AI Act, Article 14), pas comme un système de décision automatisée.

### Mesures implémentées

| Exigence AI Act | Implémentation |
|-----------------|----------------|
| **Transparence** | Score décomposé par axe, justifications textuelles, explainability complète |
| **Non-discrimination** | Anonymisation côté client + instruction explicite au modèle d'ignorer tout critère protégé |
| **Contrôle humain** | Le score ≠ une décision. Mention systématique que l'humain reste décisionnaire |
| **Minimisation des données** | Aucune donnée stockée. Clé API effacée à la fermeture de l'onglet |
| **Auditabilité** | Pondération affichée, prompt système documenté, version taguée |

### Données anonymisées avant envoi

L'anonymisation côté client supprime automatiquement :

- Noms et prénoms (pattern Prénom NOM)
- Civilités (M., Mme, Mlle)
- Emails, téléphones, adresses, codes postaux
- Dates de naissance, âge, lieu de naissance
- Nationalité, situation familiale, nombre d'enfants
- Mentions de handicap, RQTH, AAH, MDPH
- Numéro de sécurité sociale, IBAN
- URLs, profils LinkedIn et réseaux sociaux
- Références à une photo

---

## Sécurité — Audit v2.2-sec

La version 2.2 corrige les 7 vulnérabilités identifiées par l'audit de sécurité SGPT.

### Vulnérabilités corrigées

| # | Sévérité | Vulnérabilité | Correctif |
|---|----------|---------------|-----------|
| 1 | 🔴 CRITIQUE | XSS via `innerHTML` avec `file.name` non sanitisé | `renderFileLoaded` réécrit en DOM pur (`createElement` + `textContent`) |
| 2 | 🔴 CRITIQUE | Google Fonts sans Subresource Integrity | Ajout `crossorigin="anonymous"` + `referrerpolicy="no-referrer"` + preconnect |
| 3 | 🟠 ÉLEVÉE | Absence de Content Security Policy | Meta CSP stricte : `default-src 'self'`, `connect-src` limité à `api.openai.com`, `frame-ancestors 'none'` |
| 4 | 🟠 ÉLEVÉE | Clé API en mémoire sans protection | Encodage base64, getter/setter opaque, effacement au `beforeunload`, validation format `sk-`, champ vidé après save |
| 5 | 🟠 ÉLEVÉE | Absence d'en-têtes de sécurité HTTP | `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy` |
| 6 | 🟡 MOYENNE | Anonymisation insuffisante | +10 patterns : URLs, LinkedIn, n° sécu, IBAN, enfants, noms propres, unicode étendu |
| 7 | 🟡 MOYENNE | `innerHTML` non sanitisé dans `renderResults` | Fonction `esc()` appliquée à tout contenu provenant de l'API. `safeNum()` pour les valeurs numériques injectées dans les styles |

### Mesures complémentaires

- `"use strict"` activé globalement
- Badge visuel **CSP + XSS** dans le header pour confirmer les protections actives
- Avertissement dans les paramètres rappelant qu'en production, un proxy backend est nécessaire

---

## Utilisation

### Prérequis

- Un navigateur moderne (Chrome, Firefox, Edge, Safari)
- Une clé API OpenAI avec accès à GPT-4o ou GPT-4.1

### Lancement

1. Ouvrir `matchsense-v2.1-sec.html` dans un navigateur
2. Cliquer sur **Paramètres** → saisir la clé API OpenAI → **Enregistrer**
3. Déposer la **fiche de poste** (drag & drop ou clic) — formats : `.pdf`, `.txt`, `.docx`
4. Renseigner les **valeurs et culture d'entreprise** dans la zone de texte
5. Déposer les **3 CV** des candidats A, B et C
6. Cliquer sur **C'est parti ?**

### Résultats

- **Podium comparatif** : classement 1/2/3 avec score global et barres par axe
- **Recommandation comparative** : synthèse globale en prose
- **Fiches détaillées** : par candidat, avec justification de chaque axe, signaux faibles détectés, points forts et points de vigilance
- **Disclaimer AI Act** : rappel que l'outil est une aide à la décision

---

## Jeu de données de test

Trois CV fictifs sont fournis pour tester le comportement de l'outil sur des profils contrastés.

### CV n°2 — Kévin Morel (`CV2_Kevin_Morel_Maintenance.txt`)

**Profil** : ultra technique, instable, faible relationnel

- 14 postes en 12 ans, majorité en intérim de 2 à 6 mois
- Compétences techniques excellentes (automates, hydraulique, soudure, GMAO, habilitations)
- Red flags relationnels explicites dans le CV (« difficultés d'intégration », « communication insuffisante »)
- Licenciement suite à un différend avec un chef d'équipe
- Zéro engagement associatif, centres d'intérêt solitaires

**Score attendu** : technique haut (~18-20/20), valeurs/engagement très bas, soft skills bas → score global ~35-50.

### CV n°3 — Yassine Benmoussa (`CV3_Yassine_Benmoussa.docx`)

**Profil** : bon technicien, stable, engagé, avec un trou de 2 ans

- 4 CDI longs (jusqu'à 4 ans 6 mois chez Airbus Atlantic)
- Trou de 2 ans (2019-2021) — « raisons personnelles et familiales »
- Signaux forts d'engagement : formateur, tuteur d'alternants, délégué sécurité, participant TPM
- Bénévolat : soutien scolaire, coach foot enfants, courses caritatives
- Nom à consonance arabe → teste que l'anonymisation neutralise tout biais

**Score attendu** : valeurs/engagement haut (~32-38/40), soft skills haut, technique bon → score global ~70-85. Le trou dans le CV ne doit pas être sur-pénalisé.

### CV n°1 — (à créer)

Profil suggéré : équilibré, technicien correct, bon relationnel, parcours classique. Sert de baseline.

---

## Architecture technique

```
matchsense-v2.1-sec.html     ← Application complète (single-file HTML)
├── <head>
│   ├── Meta CSP + en-têtes sécurité
│   ├── Google Fonts (crossorigin)
│   └── <style> — design system complet
├── <body>
│   ├── Header (logo, badges AI Act + CSP, paramètres)
│   ├── Workspace
│   │   ├── Drop zone fiche de poste
│   │   ├── Zone valeurs/culture
│   │   ├── 3× Drop zones CV (A, B, C)
│   │   └── Bouton "C'est parti ?"
│   ├── Section résultats (générée dynamiquement)
│   ├── Overlay de chargement
│   └── Modal paramètres (clé API, modèle)
└── <script>
    ├── esc() / safeNum()      — sanitisation XSS
    ├── anon()                 — anonymisation client-side (25+ patterns)
    ├── extractText()          — extraction texte PDF/TXT/DOCX
    ├── renderFileLoaded()     — affichage fichier (DOM pur, pas innerHTML)
    ├── runMatching()          — appel API OpenAI avec prompt structuré
    └── renderResults()        — rendu résultats (tout échappé via esc())
```

### Flux de données

```
Fichiers utilisateur
       │
       ▼
  extractText()          ← Extraction texte brut côté client
       │
       ▼
     anon()              ← Anonymisation (25+ regex, côté client)
       │
       ▼
  API OpenAI             ← Seules les données anonymisées sortent du navigateur
       │
       ▼
  JSON structuré         ← Scores, justifications, signaux faibles
       │
       ▼
  esc() + safeNum()      ← Sanitisation avant injection DOM
       │
       ▼
  Rendu résultats        ← Affichage sécurisé
```

---

## Modèles supportés

| Modèle | Recommandation | Notes |
|--------|---------------|-------|
| `gpt-4o` | ✅ Recommandé | Meilleur rapport qualité/coût pour l'analyse de CV |
| `gpt-4o-mini` | ⚡ Rapide | Moins précis sur les signaux faibles |
| `gpt-4.1` | 🧪 Nouveau | À tester — potentiellement meilleur sur le raisonnement |
| `gpt-4.1-mini` | ⚡ Rapide | Alternative rapide de dernière génération |

---

## Limites connues

- **Extraction PDF/DOCX basique** : l'extraction texte côté client est rudimentaire (lecture binaire des caractères imprimables). Les PDF complexes ou les DOCX avec mise en page avancée peuvent perdre du contenu. Pour de meilleurs résultats, utiliser des fichiers `.txt`.
- **Pas de proxy backend** : la clé API OpenAI transite côté client. Acceptable pour un prototype interne, mais en production il faut un backend intermédiaire.
- **Anonymisation par regex** : couverture large (25+ patterns) mais pas infaillible. Des prénoms rares ou des formulations inhabituelles peuvent passer.
- **Dépendance au modèle** : la qualité du scoring dépend du modèle OpenAI utilisé. Les résultats peuvent varier entre deux exécutions (température à 0.15 pour limiter la variance).

---

## Roadmap

- [ ] CV n°1 de test (profil baseline équilibré)
- [ ] Fiche de poste de test (maintenance industrielle)
- [ ] Proxy backend pour sécuriser la clé API
- [ ] Extraction PDF via pdf.js côté client
- [ ] Export des résultats en PDF
- [ ] Mode comparaison 2 ou 5 candidats
- [ ] Historique local des analyses (chiffré)
- [ ] Tests A/B sur la pondération des axes

---

## Stack

- **Frontend** : HTML/CSS/JS vanilla — single-file, zéro dépendance
- **IA** : API OpenAI (GPT-4o / GPT-4.1) via `fetch`
- **Sécurité** : CSP, XSS sanitization, anonymisation client-side, en-têtes HTTP

---

## Licence

Outil interne — usage restreint au périmètre de l'organisation.

---

*MatchSense v2.1-sec — Conçu pour trouver le signal dans le bruit.*
