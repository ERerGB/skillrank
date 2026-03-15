# ADR-002: D1 Database Schema Design

Status: Proposed
Date: 2026-02-28

## Context

SkillRank needs persistent storage for hub registry, crawled skill metadata,
ranking signals, and rank history. D1 (SQLite) is the primary database.

## Decision

### Tables

```sql
-- Hub registry: the primary ranked entities
CREATE TABLE hubs (
  id          TEXT PRIMARY KEY,          -- e.g. "clawhub", "skillkit"
  name        TEXT NOT NULL,             -- Display name
  url         TEXT NOT NULL,             -- Hub homepage URL
  api_url     TEXT,                      -- API endpoint for crawling
  sdk_package TEXT,                      -- npm/pip package name if available
  created_at  TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Skill metadata: crawled from hubs (metadata under each hub)
CREATE TABLE skills (
  id          TEXT PRIMARY KEY,          -- hub_id + ":" + skill_slug
  hub_id      TEXT NOT NULL REFERENCES hubs(id),
  slug        TEXT NOT NULL,             -- Skill identifier within hub
  name        TEXT NOT NULL,
  description TEXT,
  author      TEXT,
  version     TEXT,
  install_count INTEGER DEFAULT 0,
  tags        TEXT,                      -- JSON array of tags
  crawled_at  TEXT NOT NULL DEFAULT (datetime('now')),
  UNIQUE(hub_id, slug)
);

-- Signals: feedback from Prism (V1: store only, V2: use in ranking)
CREATE TABLE signals (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  hub_id      TEXT NOT NULL REFERENCES hubs(id),
  type        TEXT NOT NULL,             -- citation_density, contributor_reputation, etc.
  value       REAL NOT NULL,
  metadata    TEXT,                      -- JSON blob
  source      TEXT NOT NULL DEFAULT 'prism',
  created_at  TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE INDEX idx_signals_hub ON signals(hub_id, type);

-- Rank snapshots: historical hub scores
CREATE TABLE rank_history (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  hub_id      TEXT NOT NULL REFERENCES hubs(id),
  score       REAL NOT NULL,
  components  TEXT,                      -- JSON: { api_freshness, skill_count, ... }
  computed_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE INDEX idx_rank_history_hub ON rank_history(hub_id, computed_at);

-- Crawl logs: track crawler runs
CREATE TABLE crawl_logs (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  hub_id      TEXT NOT NULL REFERENCES hubs(id),
  status      TEXT NOT NULL,             -- success, error, timeout
  skills_found INTEGER DEFAULT 0,
  duration_ms INTEGER,
  error       TEXT,
  crawled_at  TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### Initial Hub Seeds

```sql
INSERT INTO hubs (id, name, url, sdk_package) VALUES
  ('clawhub', 'ClawHub', 'https://clawhub.com', 'clawhub'),
  ('skillkit', 'SkillKit Marketplace', 'https://skillkit.dev', 'skillkit'),
  ('cursor', 'Cursor Marketplace', 'https://cursor.com', NULL);
```

## Consequences

### Benefits
- Simple, flat schema — easy to query and extend
- Signals table is append-only — safe for V1 "store only" strategy
- Rank history enables trend analysis and debugging

### Costs
- Tags as JSON string requires parsing in application layer
- No foreign key enforcement in D1 by default (must enable via PRAGMA)
