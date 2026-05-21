<p align="center">
  <img src="https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-logo.png" width="160" alt="n8n" />
</p>
<h1 align="center">FlowMind — AI Delivery Intelligence Airtable</h1>
 
<div align="center">

[![n8n](https://img.shields.io/badge/n8n-Orchestration-FF6D5A?logo=n8n&logoColor=white)](https://n8n.io)
[![Google Gemini](https://img.shields.io/badge/LLM-Google%20Gemini-4285F4?logo=google&logoColor=white)](https://ai.google.dev)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-4169E1?logo=postgresql&logoColor=white)](https://postgresql.org)
[![Airtable](https://img.shields.io/badge/Source-Airtable-18BFFF?logo=airtable&logoColor=white)](https://airtable.com)
[![Slack](https://img.shields.io/badge/Slack-Notifications-4A154B?logo=slack&logoColor=white)](https://slack.com)
[![Multi-tenant](https://img.shields.io/badge/Multi--tenant-Ready-A855F7)]()
[![Nodes](https://img.shields.io/badge/Nodes-117-22C55E)]()
[![Status](https://img.shields.io/badge/Status-Production%20Ready-22C55E)]()
[![License](https://img.shields.io/badge/License-MIT-22C55E)]()

**Système d'orchestration décisionnelle IA pour la prévention d'échec de sprint logiciel**

*Détecte · Analyse · Décide · Agit — sans attendre*

</div>

## Image Workflow

<p align="center">
  <img src="https://github.com/Tchatchoua14/FlowMind-AI-Delivery-Intelligence-Airtable/blob/c53fbf0821d42daa44354a2e074cc5b1ae9c322c/Capture%20d%E2%80%99%C3%A9cran%202026-05-20%20175006.png" alt="n8n Logo" />
</p>

---
 
## Vue d'ensemble
 
**FlowMind** est un système d'orchestration décisionnelle construit sur **n8n**, propulsé par **Google Gemini**, qui surveille en continu l'état d'un sprint logiciel sur Airtable, calcule un score de santé, détecte les risques, lance 4 agents IA spécialisés en parallèle et produit des décisions opérationnelles actionnables — avec une mémoire inter-sprints qui calcule automatiquement les tendances.
 
Le système couvre l'intégralité du cycle décisionnel :
 
- **Collecte** — Airtable API, normalisée dans un modèle unifié tâche / sprint
- **Charge développeur** — détection automatique de surcharge par assignee
- **Scoring** — `risk_score` (0–100+) par tâche + `sprint_health_score` global (0–100)
- **Mémoire** — historique des snapshots inter-sprints avec calcul de tendance (dégradation / stable / amélioration)
- **Analyse IA** — 4 agents Gemini spécialisés raisonnent en parallèle sur le contexte enrichi
- **Synthèse** — un agent orchestrateur produit la décision finale avec résumé, escalades et actions prioritaires
- **Action** — notification Slack filtrée par seuil de confiance (≥ 60%) et cooldown anti-flood (6h)
- **Traçabilité** — toutes les décisions, erreurs et snapshots sont persistés dans PostgreSQL
> FlowMind ne recommande pas. Il décide. La supervision humaine reste la finalité — mais le chemin jusqu'à l'humain est entièrement automatisé.
 
---
 
## Table des matières
 
- [Architecture cognitive](#architecture-cognitive)
- [Pipeline décisionnel](#pipeline-décisionnel)
  - [Étape 1 — Collecte et normalisation](#étape-1--collecte-et-normalisation)
  - [Étape 2 — Charge développeur](#étape-2--charge-développeur)
  - [Étape 3 — Scoring de risque](#étape-3--scoring-de-risque)
  - [Étape 4 — Mémoire inter-sprints](#étape-4--mémoire-inter-sprints)
  - [Étape 5 — Agents IA spécialisés](#étape-5--agents-ia-spécialisés)
  - [Étape 6 — Synthèse décisionnelle](#étape-6--synthèse-décisionnelle)
  - [Étape 7 — Notifications intelligentes](#étape-7--notifications-intelligentes)
- [Modèle de scoring](#modèle-de-scoring)
- [Modèle de données](#modèle-de-données)
- [Gestion des erreurs](#gestion-des-erreurs)
- [Installation](#installation)
- [Roadmap](#roadmap)
- [Chiffres clés](#chiffres-clés)
---
 
## Architecture cognitive
 
```
┌──────────────────────────────────────────────────────────┐
│                   SOURCE DE DONNÉES                      │
│                       Airtable                           │
│               (extensible : Jira · Linear)               │
└──────────────────────────┬───────────────────────────────┘
                           │ HTTP API + normalisation
                    ┌──────▼──────┐
                    │  MODÈLE     │  Format unifié
                    │  UNIFIÉ     │  task · sprint · assignee
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  DEVELOPER  │  Charge par assignee
                    │  LOAD       │  Surcharge détectée
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  RISK       │  Score 0–100+ par tâche
                    │  ENGINE     │  Sprint Health Score global
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  SPRINT     │  Snapshot historique
                    │  MEMORY     │  Tendance inter-sprints
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼──────┐  ┌──────▼──────┐  ┌─────▼──────┐
    │ Guardian   │  │  Friction   │  │ Knowledge  │
    │ of         │  │  Killer     │  │ Gap        │
    │ Deadlines  │  │             │  │ Detector   │
    └─────┬──────┘  └──────┬──────┘  └─────┬──────┘
          └────────────────┼────────────────┘
                           │ Merge — Agent Outputs
                    ┌──────▼──────┐
                    │  DECISION   │  Decision Synthesizer
                    │  SYNTHESIZER│  Gemini — JSON structuré
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼──────┐  ┌──────▼──────┐  ┌─────▼──────┐
    │  Postgres  │  │  Slack PM   │  │  Failed    │
    │  Decisions │  │  Notify     │  │  Jobs Log  │
    └────────────┘  └─────────────┘  └────────────┘
```
 
### Stack technique
 
| Composant | Rôle | Détail |
|-----------|------|--------|
| **n8n** | Orchestration | Workflow complet, gestion d'erreurs, retry |
| **Google Gemini** | LLM décisionnel | 4 agents en parallèle via LangChain |
| **PostgreSQL** | Persistance | 7 tables métier |
| **Airtable** | Source sprint | Tâches, statuts, assignees, blocages |
| **Slack** | Notifications | PM + Ops avec anti-flood 6h |
 
---
 
## Pipeline décisionnel
 
### Étape 1 — Collecte et normalisation
 
Déclenchement par **Schedule Trigger** (intervalle configurable, minimum recommandé : 5 min).
 
```
Set - Client Context
  └─→ HTTP - Airtable Get Sprint Items
        ├─ SUCCESS → Code - Normalize Airtable Sprint Items
        └─ ERROR   → Set - Airtable API Error
                        └─→ Postgres - Insert API Audit Log
                              └─→ Postgres - Insert Failed Job
                                    └─→ Slack - Notify Ops API Error
                                          └─→ Wait 2 min
                                                └─→ IF retry < 3 → relancer
```
 
**Validation stricte** : `type === "task"` ET `external_id !== ""` ET `title !== ""`.
Tout élément invalide est journalisé dans `audit_logs` avec `error_type: VALIDATION_ERROR`.
 
**Normalisation** : le nœud `Code - Normalize Airtable Sprint Items` mappe les colonnes Airtable en format unifié, gère le français et l'anglais (État / Status / Statut, Assignée à / Responsable / Owner...) et détecte automatiquement les blocages par analyse textuelle.
 
```javascript
// Champs normalisés
{
  source: "airtable",
  type: "task",
  external_id: "rec_XXXX",
  title: "Implémenter le module paiement",
  status: "En cours",
  assignee: "Alice",
  assignee_email: "alice@co.com",
  priority: "Haute",            // normalisé : Critique / Haute / Moyenne / Basse
  sprint: "Sprint 3",
  blocked: true,
  blocker_type: "Validation",   // Validation / Technique / Fonctionnel / Dépendance
  sprint_start_date: "2026-05-20",
  sprint_end_date: "2026-06-03",
  age_days: 4
}
```
 
---
 
### Étape 2 — Charge développeur
 
Le nœud `Code - Compute Developer Load` analyse l'ensemble des tâches normalisées et calcule la charge de chaque assignee **avant** le scoring individuel.
 
```javascript
// Critères de surcharge (developer_overloaded = true)
total_open_tasks >= 6
|| blocked_tasks >= 3
|| high_priority_tasks >= 4
```
 
Chaque tâche reçoit en sortie :
 
```javascript
developer_load: {
  total_open_tasks: 7,
  blocked_tasks: 2,
  high_priority_tasks: 5
},
developer_overloaded: true,
developer_load_summary: "Alice a 7 tâche(s) ouvertes, 2 bloquée(s), 5 prioritaire(s)."
```
 
---
 
### Étape 3 — Scoring de risque
 
Le nœud `Code - Detect Delivery Risks` calcule un `risk_score` pour chaque tâche valide.
 
#### Signaux détectés
 
| Signal | Points |
|--------|--------|
| Élément bloqué (`blocked = true`) | +30 |
| CI/CD échouée | +25 |
| Deadline dépassée | +25 |
| Review ou validation en attente | +20 |
| Fin de sprint imminente (≤ 1 jour) | +20 |
| Développeur surchargé | +20 |
| Aucun responsable assigné | +15 |
| Moins de 3 jours restants dans le sprint | +12 |
| Inactivité ≥ 3 jours | +10 |
| Tâche ouverte depuis ≥ 5 jours | +10 |
| Priorité critique | +20 |
| Priorité haute | +15 |
| Priorité moyenne | +8 |
 
**Seuil de filtrage** : seuls les éléments avec `risk_score >= 30` entrent dans le pipeline décisionnel.
 
#### Niveaux de risque
 
| Niveau | Score |
|--------|-------|
| Faible | < 30 |
| Moyen | 30–54 |
| Élevé | 55–79 |
| Critique | ≥ 80 |
 
#### Sprint Health Score (0–100)
 
```javascript
sprintHealthScore = 100
  - (criticalRisks.length × 18)
  - (highRisks.length × 10)
  - (mediumRisks.length × 5)
  - (totalRiskScore >= 300 ? 20 : totalRiskScore >= 200 ? 12 : totalRiskScore >= 100 ? 6 : 0)
 
// Plancher à 0
sprintHealthScore = Math.max(0, sprintHealthScore)
```
 
| Sprint Health | Niveau global |
|---------------|---------------|
| ≥ 80 | Faible |
| 60–79 | Moyen |
| 40–59 | Élevé |
| < 40 | Critique |
 
---
 
### Étape 4 — Mémoire inter-sprints
 
FlowMind stocke un snapshot de santé à chaque exécution et calcule automatiquement la tendance par rapport à la mesure précédente.
 
```
Code - Sprint Health Score
  └─→ Code - Build Single Sprint Health Snapshot
        └─→ Set - Current Sprint Health Context
              └─→ Postgres - Insert Sprint Health Snapshot
                    └─→ Postgres - Get Previous Sprint Health
                          └─→ Code - Calculate Sprint Trend
```
 
#### Calcul de tendance
 
```javascript
trend_delta = currentScore - previousScore
 
// Seuils
if (trend_delta <= -10) trend = "degradation"   // Alerte renforcée
if (trend_delta >= 10)  trend = "amelioration"  // Signal positif
else                    trend = "stable"
```
 
Exemple de sortie :
 
```javascript
{
  trend: "degradation",
  trend_label: "Dégradation",
  trend_delta: -14,
  trend_message: "Le sprint se dégrade : -14 points par rapport à la précédente mesure.",
  previous_sprint_health_score: 72,
  current_sprint_health_score: 58,
  previous_analyzed_at: "2026-05-19T..."
}
```
 
---
 
### Étape 5 — Agents IA spécialisés
 
Le nœud `Code - Build Anti-Failure Context` prépare un objet structuré et l'envoie **en parallèle** aux 3 agents Gemini.
 
Chaque agent dispose de :
- `retryOnFail: true` avec `waitBetweenTries: 5000ms`
- `onError: continueErrorOutput` → fallback JSON structuré si Gemini est indisponible
- Journalisation dans `failed_jobs` en cas d'échec
#### Agent 1 — Guardian of Deadlines
 
> Prédit le risque d'échec de sprint et recommande des arbitrages de scope.
 
```json
{
  "agent": "Guardian of Deadlines",
  "sprint_failure_probability": 72,
  "delivery_forecast": "Livraison compromise si blocage non levé avant J-2",
  "global_risk_level": "Élevé",
  "main_failure_causes": ["2 tâches critiques sans owner", "Dev surchargé sur TASK-07"],
  "scope_decisions": [],
  "critical_actions": [
    {
      "action": "Débloquer TASK-07 en urgence",
      "owner": "Tech Lead",
      "deadline": "Aujourd'hui 17h",
      "reason": "Bloque 3 autres tâches en cascade"
    }
  ],
  "tasks_to_deprioritize": ["TASK-12", "TASK-15"],
  "tasks_to_protect": ["TASK-07", "TASK-03"],
  "confidence_score": 82,
  "critical_decision": true
}
```
 
#### Agent 2 — Friction Killer
 
> Analyse les temps morts de validation : revues absentes, blocages de process, tâches sans owner.
 
```json
{
  "agent": "Friction Killer",
  "friction_level": "Élevé",
  "blocked_reviews": ["TASK-04 — validation client bloquée depuis 3 jours"],
  "slow_reviews": [],
  "items_without_owner": ["TASK-11"],
  "recommended_review_actions": [
    {
      "target": "TASK-04",
      "action": "Relancer le client pour décision go/no-go",
      "owner": "PM",
      "urgency": "Immédiate"
    }
  ],
  "review_acceleration_summary": "2 tâches nécessitent une intervention externe immédiate.",
  "confidence_score": 75,
  "critical_decision": false
}
```
 
#### Agent 3 — Knowledge Gap Detector
 
> Identifie les manques de compétence, les dépendances critiques à une seule personne et la documentation manquante.
 
```json
{
  "agent": "Knowledge Gap Detector",
  "knowledge_gap_level": "Moyen",
  "knowledge_risks": ["TASK-09 : dépendance unique sur Bob (seul à maîtriser l'API paiement)"],
  "senior_needed_topics": ["Architecture microservices", "Intégration Stripe"],
  "documentation_needed": ["Guide d'intégration API paiement", "Runbook déploiement"],
  "recommended_knowledge_actions": [
    {
      "target": "TASK-09",
      "action": "Session de pair programming Bob → Alice pour transfert de connaissance",
      "owner": "Tech Lead",
      "urgency": "Avant demain 14h"
    }
  ],
  "confidence_score": 68,
  "critical_decision": false
}
```
 
---
 
### Étape 6 — Synthèse décisionnelle
 
Les 3 sorties agents sont mergées dans `Merge - Agent Outputs`, puis transmises au **Decision Synthesizer**.
 
En cas d'échec d'un agent, son fallback JSON (avec `agent_failed: true`) est passé au Merge — le Synthesizer sait qu'il travaille sur des données partielles et le signale dans `ai_data_quality`.
 
#### Agent 4 — Decision Synthesizer
 
> Transforme les 3 analyses en une décision unique, actionnelle, distribuable.
 
```json
{
  "final_summary": "Sprint en risque élevé. 2 tâches critiques bloquées, 1 dev surchargé.",
  "global_risk_level": "Élevé",
  "sprint_failure_probability": 68,
  "critical_decision": true,
  "ai_data_quality": {
    "complete": true,
    "failed_agents": [],
    "warning": ""
  },
  "decisions_today": [
    "Débloquer TASK-07 avant 17h — contacter Tech Lead",
    "Assigner TASK-11 à un responsable immédiatement"
  ],
  "review_actions": ["Relancer client sur TASK-04"],
  "scope_actions": ["Sortir TASK-12 du scope sprint actuel"],
  "escalations": ["Alerter le CTO si TASK-07 non débloqué avant fin de journée"],
  "notification_required": true,
  "notification_target": "pm",
  "slack_message": "Sprint à risque élevé (68% échec). 2 décisions urgentes aujourd'hui.",
  "confidence_score": 81
}
```
 
**Validation de la sortie** (`Code - Validate Final Decision Output`) :
 
- Parse le JSON avec fallback regex si Gemini retourne du texte libre
- Normalise le `confidence_score` (détecte 0–1 vs 0–100)
- En cas d'échec total : pousse un fallback opérationnel avec `decision_valid: false` et notification Ops
**Filtrage de notification** (`IF - Notification Required ?`) :
 
```javascript
// Déclenche Slack seulement si :
(notification_required === true && confidence_score >= 60)
|| decision_valid === false
|| ai_data_quality.complete === false
```
 
**Anti-flood** (`IF - Should Notify ?`) :
 
```javascript
// Pas de notification si la dernière date de moins de 6h
!last_notified_at
|| Date.now() - new Date(last_notified_at) > 6 * 60 * 60 * 1000
```
 
---
 
### Étape 7 — Notifications intelligentes
 
```
IF - Notification Required ?
  ├─ TRUE  → Set - Notify PM / Second Reviewer
  │              └─→ IF - Should Notify ?
  │                    └─ TRUE → Slack - Sprint à risque
  │                                  (risk_level · failure_prob · decisions_today)
  └─ FALSE → Postgres - Insert Agent Recommendations
                  (archivage silencieux — pas de notification)
```
 
Format du message Slack :
 
```
*FlowMind — Sprint à risque*
 
*Niveau global* : Élevé
*Probabilité d'échec* : 68%
*Confiance IA* : 81%
 
*Résumé*
Sprint en risque élevé. 2 tâches critiques bloquées, 1 dev surchargé.
 
*Message*
Sprint à risque élevé (68% échec). 2 décisions urgentes aujourd'hui.
```
 
---
 
## Modèle de scoring
 
### Algorithme de `estimated_failure_probability`
 
```javascript
// Base selon sprint health score
if (sprintHealthScore >= 80) base = 15;
else if (sprintHealthScore >= 60) base = 35;
else if (sprintHealthScore >= 40) base = 60;
else base = 80;
 
// Ajustements
probability = base
  + (criticalRisks.length × 5)
  + (highRisks.length × 3)
  + (validationRisks.length × 2);
 
// Plafond à 95%
probability = Math.min(probability, 95);
```
 
---
 
## Modèle de données
 
### `delivery_risks` — Risques détectés par tâche
 
```sql
CREATE TABLE delivery_risks (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  alert_key            TEXT UNIQUE NOT NULL,
  source               TEXT,
  type                 TEXT,
  external_id          TEXT,
  task_id              TEXT,
  title                TEXT,
  status               TEXT,
  assignee             TEXT,
  priority             TEXT,
  sprint               TEXT,
  risk_score           INTEGER,
  risk_level           TEXT,
  risk_type            TEXT,
  reasons_detected     TEXT,
  decision_required    TEXT,
  blocker_type         TEXT,
  pr_blocker_summary   TEXT,
  linked_pr_count      INTEGER DEFAULT 0,
  linked_prs           JSONB DEFAULT '[]',
  source_url           TEXT,
  sprint_health_score  INTEGER,
  global_risk_level    TEXT,
  total_risks          INTEGER,
  critical_risks       INTEGER,
  high_risks           INTEGER,
  medium_risks         INTEGER,
  executive_summary    TEXT,
  status_alert         TEXT DEFAULT 'Ouverte',
  detected_at          TIMESTAMPTZ,
  updated_at           TIMESTAMPTZ DEFAULT NOW(),
  raw_context          JSONB
);
 
CREATE UNIQUE INDEX ON delivery_risks (alert_key);
CREATE INDEX ON delivery_risks (risk_level);
CREATE INDEX ON delivery_risks (sprint);
CREATE INDEX ON delivery_risks (assignee);
CREATE INDEX ON delivery_risks (detected_at DESC);
```
 
### `delivery_decisions` — Décisions IA finales
 
```sql
CREATE TABLE delivery_decisions (
  id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id                   TEXT,
  client_name                 TEXT,
  final_summary               TEXT,
  global_risk_level           TEXT,
  sprint_failure_probability  INTEGER,
  critical_decision           BOOLEAN DEFAULT false,
  decisions_today             JSONB DEFAULT '[]',
  review_actions              JSONB DEFAULT '[]',
  scope_actions               JSONB DEFAULT '[]',
  escalations                 JSONB DEFAULT '[]',
  notification_required       BOOLEAN DEFAULT false,
  notification_target         TEXT,
  confidence_score            INTEGER DEFAULT 0,
  ai_data_quality             JSONB DEFAULT '{}',
  raw_decision                JSONB,
  status                      TEXT DEFAULT 'pending',
  created_at                  TIMESTAMPTZ DEFAULT NOW(),
  updated_at                  TIMESTAMPTZ DEFAULT NOW()
);
 
CREATE INDEX ON delivery_decisions (global_risk_level);
CREATE INDEX ON delivery_decisions (created_at DESC);
CREATE INDEX ON delivery_decisions (client_id);
```
 
### `sprint_health_history` — Mémoire inter-sprints
 
```sql
CREATE TABLE sprint_health_history (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id            TEXT,
  client_name          TEXT,
  sprint               TEXT,
  sprint_start_date    DATE,
  sprint_end_date      DATE,
  sprint_health_score  INTEGER,
  global_risk_level    TEXT,
  total_risks          INTEGER,
  critical_risks       INTEGER,
  high_risks           INTEGER,
  medium_risks         INTEGER,
  total_risk_score     INTEGER,
  top_risk_types       TEXT,
  top_assignees        TEXT,
  executive_summary    TEXT,
  analyzed_at          TIMESTAMPTZ DEFAULT NOW(),
  raw_context          JSONB
);
 
CREATE INDEX ON sprint_health_history (client_id, analyzed_at DESC);
CREATE INDEX ON sprint_health_history (sprint);
```
 
### `audit_logs` — Traçabilité validation et erreurs
 
```sql
CREATE TABLE audit_logs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id      TEXT,
  level       TEXT,        -- info | warn | error
  source      TEXT,
  error_type  TEXT,        -- VALIDATION_ERROR | API_FAILURE | AI_AGENT_FAILURE
  message     TEXT,
  details     JSONB,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
 
CREATE INDEX ON audit_logs (level, created_at DESC);
CREATE INDEX ON audit_logs (source);
```
 
### `failed_jobs` — Queue de relance
 
```sql
CREATE TABLE failed_jobs (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id        TEXT,
  client_id     TEXT,
  source        TEXT,
  node_name     TEXT,
  error_type    TEXT,
  error_message TEXT,
  payload       JSONB,
  retry_count   INTEGER DEFAULT 0,
  status        TEXT DEFAULT 'open',  -- open | resolved | abandoned
  created_at    TIMESTAMPTZ DEFAULT NOW()
);
 
CREATE INDEX ON failed_jobs (status, created_at DESC);
CREATE INDEX ON failed_jobs (client_id);
```
 
---
 
## Gestion des erreurs
 
### Matrice complète par couche
 
| Zone | Erreur possible | Nœud de traitement | Action |
|------|-----------------|-------------------|--------|
| API Airtable | HTTP 4xx / 5xx / timeout | `Set - Airtable API Error` | Log Postgres + Slack Ops + retry ×3 |
| Normalisation | Résultat vide | `empty_airtable_result` warning | Warning loggé, pipeline continue |
| Validation item | `external_id` ou `title` absent | `Set - Airtable Validation Error` | `audit_logs` VALIDATION_ERROR |
| Agent Guardian | Gemini timeout / JSON invalide | `Set - Guardian AI Error` | Fallback JSON + `failed_jobs` |
| Agent Friction | Gemini timeout / JSON invalide | `Set - Friction Killer AI Error` | Fallback JSON + `failed_jobs` |
| Agent Knowledge | Gemini timeout / JSON invalide | `Set - Knowledge Gap AI Error` | Fallback JSON + `failed_jobs` |
| Decision Synthesizer | JSON non parseable | `Code - Validate Final Decision Output` | Fallback opérationnel + notification Ops |
| Decision Synthesizer | Gemini indisponible | `Set - Decision Synthesizer Error` | `failed_jobs` + décision de fallback |
| Postgres INSERT | Connexion perdue | Pas de handler actuel | ⚠️ À ajouter — voir roadmap |
| Slack PM | Token expiré | Pas de handler actuel | ⚠️ À ajouter — voir roadmap |
 
### Circuit de retry Airtable
 
```
HTTP - Airtable Get Sprint Items [ERREUR]
  └─→ Set - Airtable API Error
        └─→ Postgres - Insert API Audit Log
              └─→ Postgres - Insert Failed Job
                    └─→ Slack - Notify Ops API Error
                          └─→ Wait 2 min
                                └─→ IF retry_count < 3
                                      ├─ TRUE  → Set - Client Context → relance HTTP
                                      └─ FALSE → abandon (max 3 tentatives)
```
 
### Fallback agent IA
 
Chaque agent dispose de son propre fallback JSON structuré transmis au Merge — garantissant que le Decision Synthesizer reçoit toujours 3 inputs, même partiels :
 
```javascript
// Exemple fallback Guardian of Deadlines
{
  agent: "Guardian of Deadlines",
  agent_failed: true,
  confidence_score: 0,
  error_type: "AI_AGENT_FAILURE",
  output: {
    sprint_failure_probability: 0,
    delivery_forecast: "Analyse indisponible",
    global_risk_level: "À vérifier",
    critical_actions: [],
    critical_decision: false
  }
}
```
 
---
 
## Installation
 
### Prérequis
 
- n8n ≥ 1.40 (cloud ou self-hosted)
- PostgreSQL ≥ 15
- Credentials : Google Gemini API, Airtable Personal Access Token, Slack
### 1. Importer le workflow
 
Dans n8n → **Settings > Import from file** → `flowmind_delivery_intelligence.json`
 
### 2. Configurer les credentials
 
| Credential | Nœuds concernés |
|-----------|-----------------|
| `Postgres account` | Tous les nœuds Postgres |
| `Google Gemini (PaLM) Api account` | 4 nœuds LLM Gemini |
| `Slack account` | Slack Notifications + Slack Notify Ops |
 
### 3. Initialiser la base de données
 
```sql
-- Exécuter dans l'ordre
CREATE TABLE delivery_risks       (...); -- voir modèle de données
CREATE TABLE delivery_decisions   (...);
CREATE TABLE sprint_health_history(...);
CREATE TABLE audit_logs           (...);
CREATE TABLE failed_jobs          (...);
```
 
### 4. Configurer le contexte client
 
Dans le nœud `Set - Client Context`, adapter :
 
```json
{
  "client_id": "votre_client_id",
  "client_name": "Nom du client",
  "airtable_base_id": "appXXXXXXXXXXXXXX",
  "airtable_table_name": "Sprint 0",
  "airtable_token": "patXXXXXXX",
  "sprint_start_date": "2026-05-20",
  "sprint_end_date": "2026-06-03",
  "slack_channel": "#votre-channel",
  "email_to": "pm@votre-domaine.com",
  "critical_threshold": 80,
  "high_threshold": 55,
  "medium_threshold": 30
}
```
 
> **Sécurité** : ne jamais committer le token Airtable dans le JSON. Utiliser les credentials n8n ou une variable d'environnement `N8N_AIRTABLE_TOKEN`.
 
### 5. Activer le workflow
 
Settings > Activate — le Schedule Trigger démarre la collecte selon l'intervalle configuré.
 
**Intervalle recommandé** : 5 à 10 minutes (éviter les secondes — rate limit Airtable et coût Gemini).
 
---
 
## Roadmap
 
| Priorité | Fonctionnalité | Statut |
|----------|----------------|--------|
| P0 | Corriger faute SQL `audit_logss` → `audit_logs` | ⚠️ À faire avant prod |
| P0 | Retirer le token Airtable du nœud HTTP (credentials n8n) | ⚠️ Sécurité critique |
| P0 | Passer le Schedule de secondes à minutes | ⚠️ Rate limit |
| P0 | Réactiver Agent Guardian of Deadlines (`disabled: true`) | ⚠️ 1 agent sur 3 muet |
| P0 | Reconnecter `Postgres - Upsert Delivery Risks` au pipeline IA | ⚠️ Connexions vides |
| P1 | Error branch sur Slack PM et Postgres Insert Delivery Decision | Important |
| P1 | Multi-tenant via table `clients` Postgres (client_id dynamique) | Important |
| P1 | Dashboard Metabase sur `delivery_risks` + `sprint_health_history` | Visibilité produit |
| P2 | Webhooks Airtable en remplacement du polling (réactivité < 30s) | Différenciateur fort |
| P2 | Rapport PDF hebdomadaire automatique (vendredi 17h → CODIR) | Valeur C-level |
| P2 | Agent Sprint Velocity Analyzer (4e agent — vélocité vs moyenne historique) | Valeur ajoutée |
| P3 | Connecteurs Jira, Linear, GitHub Issues | Extension multi-source |
| P3 | Rate limit management Gemini (Wait entre agents en charge) | Robustesse |
| P3 | pgvector — mémoire sémantique des décisions passées | Intelligence long terme |
 
---
 
## Chiffres clés
 
| Métrique | Valeur |
|----------|--------|
| Agents IA Gemini | 4 |
| Tables PostgreSQL | 5 |
| Nœuds de gestion d'erreurs | 12+ |
| Retry automatique API | 3 tentatives × 2 min |
| Cooldown notification Slack | 6 heures |
| Seuil confiance notification | 60% minimum |
| Tendance inter-sprints | Calculée automatiquement |
| Score de risque max observable | ~225 (tous signaux cumulés) |
 
---
 
<div align="center">
Construit avec [n8n](https://n8n.io) · [Google Gemini](https://ai.google.dev) · [PostgreSQL](https://postgresql.org) · [Airtable](https://airtable.com) · [Slack](https://slack.com)
 
**FlowMind — parce que les sprints échouent en silence, pas en fanfare.**
 
</div>
