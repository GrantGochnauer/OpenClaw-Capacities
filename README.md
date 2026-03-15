# OpenClaw Capacities

A lightweight **OpenClaw skill** for real-time **Capacities** lookup, type-aware ranking, and direct `capacities://` deep links.

It uses the **current public Capacities API** as it exists today, focusing on what is reliable now: smart object lookup, useful ranking, and fast handoff back into Capacities.

## Quick Start

```bash
# 1) Clone the repo
git clone git@github.com:GrantGochnauer/OpenClaw-Capacities.git
cd OpenClaw-Capacities

# 2) Add the skill to your OpenClaw workspace
mkdir -p ~/.openclaw/workspace/skills
cp -R skills/capacities-lookup ~/.openclaw/workspace/skills/

# 3) Configure Capacities access
export CAPACITIES_API_TOKEN='your-token-here'
export CAPACITIES_SPACE_ID='your-space-id'

# 4) Sync structure metadata
python3 skills/capacities-lookup/scripts/capacities_cli.py sync-structures

# 5) Run a lookup
python3 skills/capacities-lookup/scripts/capacities_cli.py lookup "Find my notes on recovery"
```

Start a **new OpenClaw session** after installing the skill so it gets picked up.

## What it does

- Searches Capacities for likely object matches by title/search term
- Returns direct `capacities://` deep links
- Enriches results with object types using cached `/space-info` metadata
- Supports **type-aware lookup** across object types in your space
  - examples: `people`, `notes`, `meetings`, `projects`, and custom types from your Capacities structures
- Supports **intent-aware ranking**
  - example: `Find my notes on recovery protocols` prefers **Note/Reference** results over generic **Tag** hits
- Expands to a small set of **related terms** for better recall
- Tells you clearly when the **requested object type was not found** and offers sensible fallback objects

## What it does not do

Because of the current public Capacities API limitations, this skill does **not**:

- read full object bodies/content
- inspect object properties or relation fields
- traverse linked objects or reverse references
- answer membership/relationship questions unless the target object is discoverable directly from title lookup
- maintain a full local mirror or background sync loop of Capacities content

## When to use it

Use this skill when you want to:

- find the right Capacities object quickly
- jump back into Capacities with a deep link
- disambiguate likely notes, meetings, projects, people, or references by title
- help an assistant suggest likely Capacities matches during conversation

## When not to use it

This is **not** the right tool for:

- full PKM ingestion
- semantic search over all note bodies
- organization membership inference from relations
- graph traversal over linked objects
- offline full-workspace querying

## Why these limitations exist

The documented public Capacities API currently gives us enough to work with:

- spaces
- space structure metadata (`/space-info`)
- title/search-term lookup (`/lookup`)
- a couple write endpoints

But it does **not** expose a documented public way to:

- fetch full object content by id
- read object properties/relations
- perform full-content search
- incrementally sync all objects

So this skill focuses on what is reliable today: **smart lookup + type awareness + deep linking**.

## Installation

### Option 1: install locally into an OpenClaw workspace

Clone this repo, then copy or symlink the skill into your OpenClaw workspace `skills/` directory.

```bash
git clone git@github.com:GrantGochnauer/OpenClaw-Capacities.git
cd OpenClaw-Capacities
mkdir -p ~/.openclaw/workspace/skills
cp -R skills/capacities-lookup ~/.openclaw/workspace/skills/
```

Then start a **new OpenClaw session** so the skill is picked up.

### Option 2: publish/install via ClawHub later

Once published, installation can be done with:

```bash
clawhub install capacities-lookup
```

## Configuration

### Required

Set your Capacities API token:

```bash
export CAPACITIES_API_TOKEN='your-token-here'
```

Set your Capacities space id:

```bash
export CAPACITIES_SPACE_ID='your-space-id'
```

### Optional

You can also set:

```bash
export CAPACITIES_DEFAULT_RESULT_LIMIT='10'
export CAPACITIES_LOOKUP_CACHE_TTL_SECONDS='86400'
export CAPACITIES_TIMEOUT_MS='15000'
export CAPACITIES_API_BASE_URL='https://api.capacities.io'
```

### Optional fallback config file

Instead of `CAPACITIES_SPACE_ID`, you may provide a local config file at:

```text
config/capacities.json
```

Example:

```json
{
  "mainSpaceId": "your-space-id",
  "apiBaseUrl": "https://api.capacities.io",
  "timeoutMs": 15000,
  "lookupCacheTtlSeconds": 86400,
  "verifySpacesOnSync": true,
  "defaultResultLimit": 10,
  "cacheSchemaVersion": 1
}
```

## Usage

### First sync structure metadata

```bash
source ~/.zshrc >/dev/null 2>&1 || true
python3 skills/capacities-lookup/scripts/capacities_cli.py sync-structures
```

### Verify the configured space

```bash
source ~/.zshrc >/dev/null 2>&1 || true
python3 skills/capacities-lookup/scripts/capacities_cli.py verify-space
```

### Run lookups

```bash
source ~/.zshrc >/dev/null 2>&1 || true
python3 skills/capacities-lookup/scripts/capacities_cli.py lookup "Find my notes on recovery"
```

```bash
source ~/.zshrc >/dev/null 2>&1 || true
python3 skills/capacities-lookup/scripts/capacities_cli.py lookup "people associated with an organization" --json
```

## Example behaviors

### Note-oriented query

Input:

```text
Find my notes on recovery protocols
```

Behavior:
- strips the type phrase (`notes on`)
- detects requested type `Note`
- searches `recovery protocols`
- may expand to related terms for better recall
- prefers Note/Reference/Page-style results over Tags

### Person-oriented query

Input:

```text
people associated with an organization
```

Behavior:
- detects requested type `Person`
- strips the type phrase
- searches for the core subject
- prefers Person objects if any exist
- if no Person objects match, reports that clearly and provides fallback objects

## Practical limitation example

The skill can often find a target **Person** object directly by name.

But it cannot currently verify whether that person is linked to some other object through a property or relation field, because the current public API does not expose object property/relationship reads.

So the skill can answer:
- “Find this person in Capacities”

But not reliably:
- “Which people are attached to this organization via relations?”

## How it works

### Live lookup first
The skill performs **real-time API lookups** when needed.

### Light cache only
It caches only:

- Capacities structures/object types from `/space-info`
- optional lookup results for convenience
- local state/error metadata

This cache exists to improve type awareness and responsiveness — not to become a second database.

## Repository layout

```text
OpenClaw-Capacities/
├── README.md
├── .gitignore
└── skills/
    └── capacities-lookup/
        ├── SKILL.md
        └── scripts/
            ├── capacities_cache.py
            ├── capacities_client.py
            ├── capacities_cli.py
            ├── capacities_lookup.py
            └── capacities_sync.py
```

## Publishing

### GitHub repo
- `https://github.com/GrantGochnauer/OpenClaw-Capacities`

### ClawHub
When ready, publish the skill folder:

```bash
clawhub publish ./skills/capacities-lookup --slug capacities-lookup --name "Capacities Lookup" --version 1.0.0 --tags latest
```

## License

MIT
