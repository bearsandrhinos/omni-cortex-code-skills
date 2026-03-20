# Omni Analytics — Cortex Code Skills

A collection of [Cortex Code](https://docs.snowflake.com/en/user-guide/cortex-code) skills for working with [Omni Analytics](https://omni.co). These skills let Cortex Code's AI agent interact with Omni's REST API to explore models, run queries, build dashboards, manage users, and more — all from your IDE.

---

## Installation

### Install from this repository

```bash
cortex skill add https://github.com/bearsandrhinos/omni-cortex-code-skills.git
```

### Or install manually

Copy the skill directories you want into your Cortex Code skills folder:

```bash
# User-level (available across all projects)
~/.snowflake/cortex/skills/

# Project-level (scoped to one project)
.cortex/skills/
```

### Verify installation

```bash
cortex skill list
```

---

## Setup

All skills use two environment variables for authentication. Set these in your shell profile (e.g. `~/.zshrc` or `~/.bashrc`):

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

**Getting your API key:**
- Personal key: Omni → Account Settings → API Keys
- Organization key (required for admin operations): Omni → Admin → API Keys

---

## Skills

### `omni-model-explorer`
Browse and inspect Omni's semantic layer — models, topics, views, fields, dimensions, measures, and relationships.

Use when you want to understand what data is available, find specific fields, check how tables are joined, or explore what topics exist before building a query or dashboard.

**Triggers:** *what can I query, what fields are available, show me the model, what data do we have, how is this data modeled*

---

### `omni-query`
Run queries against Omni's semantic layer via REST API. Supports filtering, sorting, date timeframes, and multi-step chained analysis. Can also extract and re-run queries from existing dashboards.

Use when you want to pull data, run a report, get metrics, or analyze trends.

**Triggers:** *how many, what's the trend, show me the data, run a query, pull this report*

---

### `omni-model-builder`
Create and modify Omni's semantic model — views, topics, dimensions, measures, relationships, and calculated fields — using YAML via the REST API. Always works on a branch and confirms with you before merging.

Use when you want to add fields, create new views, define joins, or modify the data model.

**Triggers:** *add this field, create a view for, set up a join between, model this data, add this metric*

---

### `omni-content-explorer`
Find, browse, and organize dashboards, workbooks, and folders in Omni. Supports label management, favorites, folder creation, PDF/PNG exports, and direct link generation.

Use when you want to find an existing dashboard, search for reports, download content, or manage labels.

**Triggers:** *find the dashboard about, what reports do we have, where is the sales report, download this dashboard*

---

### `omni-content-builder`
Create and manage Omni dashboards programmatically — documents, tiles, visualizations, filters, and layouts. Defaults to meaningful chart types (line, bar, single value, etc.) based on the data being visualized.

Use when you want to build a new dashboard, add charts, configure filters, or update an existing document.

**Triggers:** *build a dashboard for, create a report showing, add a chart to, make a dashboard*

---

### `omni-ai-optimizer`
Tune Omni's AI assistant (Blobby) by configuring `ai_context`, `ai_fields`, `sample_queries`, and AI-specific topic extensions. Helps Blobby understand your terminology, apply default filters, and answer questions accurately.

Use when Blobby is giving wrong answers, you want to improve AI accuracy, or you want to teach it about your data.

**Triggers:** *make the AI better, Blobby isn't answering correctly, add context for AI, optimize for AI, teach the AI about our data*

---

### `omni-admin`
Administer an Omni instance — manage users and groups (SCIM 2.0), assign user attributes for row-level security, set document and folder permissions, create delivery schedules, manage connections, and trigger schema refreshes.

Use when you need to provision users, set up access controls, or manage operational settings.

**Triggers:** *add a user, give access to, set up permissions, who has access, refresh the schema, schedule a delivery*

> ⚠️ Requires an **Organization API Key** — not a Personal Access Token.

---

### `omni-embed`
Embed Omni dashboards in external applications using signed URLs, custom themes, iframe event handling, and entity workspaces. Supports the `@omni-co/embed` SDK and REST API for building fully white-labeled analytics experiences.

Use when you want to embed a dashboard in your app, sign embed URLs, handle iframe events, or set up entity workspaces for multi-tenant embedding.

**Triggers:** *embed this dashboard, sign an embed URL, customize the iframe theme, handle click events from the embed, what dashboards can this user see*

---

### `omni-2-semantic-view`
Converts an Omni topic into a [Snowflake Semantic View](https://docs.snowflake.com/en/user-guide/views-semantic) YAML definition. Fetches the topic and view definitions from Omni's API, maps tables, resolves the field list (including exclusions), translates dimensions and measures, and generates a ready-to-execute `CALL SYSTEM$CREATE_SEMANTIC_VIEW_FROM_YAML(...)` statement.

Use when you want to harden Omni's metrics and definitions into the Snowflake data warehouse as a Semantic View.

**Triggers:** *Convert to Semantic View, Topic to SV, Topic to Semantic View*

---

## Skill Relationships

The skills are designed to work together:

```
omni-model-explorer  ──►  omni-query
                     ──►  omni-model-builder  ──►  omni-ai-optimizer
                     ──►  omni-2-semantic-view

omni-content-explorer  ──►  omni-content-builder
                       ──►  omni-embed

omni-admin  ──►  (manages access to all content)
```

A typical workflow might be:
1. **Explore** the model with `omni-model-explorer`
2. **Test** queries with `omni-query`
3. **Build** a dashboard with `omni-content-builder`
4. **Set permissions** with `omni-admin`
5. **Embed** it with `omni-embed`

---

## Updating Skills

```bash
cortex skill sync omni-cortex-code-skills
```

---

## Contributing

Pull requests are welcome. Each skill lives in its own directory with a single `SKILL.md` file. Skill format follows the [Cortex Code extensibility spec](https://docs.snowflake.com/en/user-guide/cortex-code/extensibility).

---

## License

Apache 2.0
