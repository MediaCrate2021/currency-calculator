# Migrate-to-Asana Tool — Planning Document

## Context

This plan outlines building a clone of [Altosio](https://altosio.com/) — a SaaS cloud migration platform for project management data. Our version is scoped specifically to **inbound migrations to Asana** from platforms like Monday.com, Smartsheet, Trello, ClickUp, etc.

**Use case:** The user frequently migrates clients/teams from other PM platforms into Asana and needs a smarter, more automated way to do it.

---

## What Altosio Does (For Reference)

Altosio is a cloud-hosted migration tool that moves boards, tasks, attachments, comments, and metadata between project management platforms. Key features:

- **Multi-platform support:** Asana, Trello, Monday.com, MS Planner, ClickUp, Smartsheet, Wrike, Todoist
- **OAuth connectors** for source and target platforms
- **Auto-discovery** of workspaces, boards, and users on connection
- **Pre-migration audit** to flag incompatibilities before running
- **User mapping** table to correctly assign tasks in the target
- **Smart field mapping** between platform-specific field types
- **Real-time migration** — reads data in chunks, pushes to target, no data stored on servers
- **Delta-pass support** — re-run migrations without duplicating data
- **Progress tracking** with real-time status, logs, and statistics
- **Per-board licensing** model (~$13.99/board)

---

## Our Scope: One Target, Multiple Sources

Instead of building a generic bidirectional tool, we're building a focused **"Migrate to Asana"** tool:

- **Target:** Always Asana (write logic built once)
- **Sources:** Multiple platforms via pluggable connectors
- **Direction:** One-way (source → Asana)

### Existing Head Start

The `currency-calculator` repo already contains an **Asana Custom Field Exporter** (`apps/custom-field-exporter/`) with:
- Asana OAuth 2.0 authentication flow
- Workspace and project discovery
- Custom field enumeration with pagination
- Rate-limit-aware API calls (concurrency control + delays)
- Express.js backend with Helmet, CORS, session management

This gives us a foundation for the Asana target connector.

---

## Source Connectors — Priority Order

| Priority | Platform | API Style | Auth | Complexity | Notes |
|----------|----------|-----------|------|------------|-------|
| 1st | **Monday.com** | GraphQL | OAuth 2.0 | Medium | Most common migration need |
| 2nd | **Smartsheet** | REST | API Token / OAuth | Low-Medium | Clean REST API, good docs |
| 3rd | **Trello** | REST | API Key + Token | Low | Simplest API |
| 4th | **ClickUp** | REST | OAuth 2.0 / Token | Medium | — |

---

## Data Model — What Gets Migrated

Per task/card, at minimum:

| Data | Source Complexity | Asana Mapping |
|------|-------------------|---------------|
| Title | Trivial | Task name |
| Description | Low | Task notes (HTML/markdown conversion may be needed) |
| Assignee(s) | Medium (Asana = 1 assignee) | Assignee + followers for extras |
| Due date | Low | Due date |
| Start date | Low | Start date |
| Completion status | Low | Completed flag |
| Comments | Medium | Story (comment) objects |
| Attachments | High (download + re-upload) | Attachment objects |
| Labels/Tags | Medium | Tags |
| Subtasks/Checklists | Medium | Subtasks |
| Custom fields | High (type mapping) | Custom fields (create or map to existing) |
| Sections/Groups | Medium | Sections |
| Board/Project structure | Medium | Projects |

---

## Architecture Decisions (To Be Confirmed)

### Open questions for the user:

1. **What does a typical migration look like?**
   - Roughly how many boards/projects per migration?
   - Are custom fields heavily used in source platforms?
   - Do you need comments and attachments, or mainly task structure?

2. **Monday.com or Smartsheet first?**
   - Which one do you encounter more often?

3. **UI preference:**
   - Full web UI (like Altosio) with step-by-step wizard?
   - Streamlined power-user interface?
   - Are you the only user, or would clients/team members use this?

4. **Field mapping complexity:**
   - Do you have standard Asana project templates you migrate into?
   - Or does the structure vary per client?

5. **User mapping:**
   - Do users typically have the same email in both platforms?
   - Or do you need a manual mapping step?

### Proposed tech stack (pending confirmation):

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Backend | Express.js | Already in use, proven in this codebase |
| Frontend | TBD (vanilla JS or React) | Depends on UI complexity needs |
| Database | PostgreSQL | Needed for migration tracking, user mapping, delta passes |
| Job queue | Bull + Redis | Migrations are long-running async jobs |
| Hosting | Railway | Already set up for existing apps |
| Auth | OAuth 2.0 per platform | Matches Altosio's connector model |

---

## Proposed Project Structure

```
apps/migration-tool/
├── server/
│   ├── connectors/              # Platform-specific read/write logic
│   │   ├── base.js              # Shared connector interface
│   │   ├── sources/
│   │   │   ├── monday.js        # Monday.com GraphQL source connector
│   │   │   ├── smartsheet.js    # Smartsheet REST source connector
│   │   │   └── trello.js        # Trello REST source connector
│   │   └── targets/
│   │       └── asana.js         # Asana write connector (extend existing)
│   ├── engine/
│   │   ├── migrator.js          # Core migration orchestration
│   │   ├── mapper.js            # Field/user mapping logic
│   │   ├── auditor.js           # Pre-migration compatibility checks
│   │   └── tracker.js           # Progress tracking & delta support
│   ├── routes/
│   │   ├── auth.js              # OAuth flows per platform
│   │   ├── discovery.js         # Workspace/board enumeration
│   │   ├── mapping.js           # User/field mapping endpoints
│   │   └── migrations.js        # Start/status/cancel/retry endpoints
│   ├── db/
│   │   ├── migrations/          # Schema migrations
│   │   └── models/              # Data models
│   └── server.js                # Express app entry point
├── public/
│   ├── index.html               # Main UI
│   ├── app.js                   # Frontend logic
│   └── styles.css               # Styling
├── package.json
└── .env.example
```

---

## MVP Scope (Phase 1)

**Goal:** One migration path, end-to-end, with a working UI.

**Pick:** [Monday.com or Smartsheet — TBD] → Asana

### MVP Features:
1. OAuth connect to source platform
2. OAuth connect to Asana (reuse existing)
3. Auto-discover workspaces/boards from source
4. Auto-discover Asana workspaces/projects as targets
5. User mapping table (auto-match by email, manual override)
6. Basic field mapping (predefined fields first, custom fields stretch goal)
7. Run migration with real-time progress
8. Success/error reporting with logs

### MVP Skips (Phase 2+):
- Pre-migration audit
- Delta-pass support
- Custom field creation/mapping
- Attachment migration
- Billing/licensing
- Multi-user support

---

## Phase 2+ Roadmap

- **Phase 2:** Add second source connector, custom field mapping, attachment migration
- **Phase 3:** Pre-migration audit, delta passes, retry failed items
- **Phase 4:** Multi-user support, billing, production hardening
- **Phase 5:** Additional source connectors (ClickUp, Wrike, etc.)

---

## Status

**Current phase:** Planning — awaiting answers to open questions before implementation begins.
