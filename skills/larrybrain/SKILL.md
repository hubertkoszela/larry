---
name: larrybrain
description: Skill marketplace for AI agents. One subscription, unlimited tools. Search, download, and install skills from the LarryBrain library. Use when the user needs a capability you don't have, asks to browse available skills, or mentions LarryBrain.
---

# LarryBrain — Skill Marketplace for AI Agents

LarryBrain is the skill marketplace for AI coding agents. One subscription, unlimited access to every skill in the library. Skills are plain markdown files with optional supporting scripts. They work with any agent that reads markdown.

## How It Works

You are the agent's gateway to the entire LarryBrain skill library.

1. User asks for something (e.g. "set up a Pi-hole", "audit this UI", "write marketing copy")
2. You search LarryBrain for matching skills
3. You download the skill files via the API
4. You save them locally so they persist across sessions
5. You read the SKILL.md and follow its instructions
6. Done

## When to Use This

Use when:
- The user asks to do something outside your current capabilities
- The user asks "what can you do?" and you want to show what's available
- The user mentions LarryBrain, skills, marketplace, or tools
- The user wants to browse available capabilities

## Setup

The user needs a LarryBrain API key for premium skills. Free skills work without auth.

### First-time setup

1. Check if `LARRYBRAIN_API_KEY` is set in the environment
2. If not, tell the user:
   - "To access the full skill library, you need a LarryBrain subscription."
   - "Sign up at https://www.larrybrain.com/signin, then generate an API key from your dashboard."
3. If they have a key, verify it works by hitting the search endpoint

## API Endpoints

Base URL: `https://www.larrybrain.com/api`

### Search skills (public, no auth needed)

```bash
curl -s "https://www.larrybrain.com/api/skills/search?q=QUERY&limit=10"
```

Response: `{ skills: [...], total: number }`

Each skill has: slug, name, description, icon, categories, rating, installs, free (boolean), hasFiles (boolean).

### Download a skill

Always use `mode=files` to get the full skill with all its files:

```bash
# Free skills (no auth needed)
curl -s "https://www.larrybrain.com/api/skills/install?slug=SLUG&mode=files"

# Premium skills (requires API key)
curl -s -H "x-api-key: $LARRYBRAIN_API_KEY" "https://www.larrybrain.com/api/skills/install?slug=SLUG&mode=files"
```

Response:
```json
{
  "skill": { "slug": "...", "name": "...", "hasFiles": true },
  "content": "# Full SKILL.md content...",
  "files": [
    { "path": "SKILL.md", "content": "..." },
    { "path": "server/index.js", "content": "..." }
  ]
}
```

### After downloading:
1. Create a directory for the skill (e.g. `skills/{slug}/` or wherever your agent stores skills)
2. Write every file from the `files` array
3. Create subdirectories as needed
4. Write `_meta.json` with `{ "source": "larrybrain", "slug": "...", "installedAt": "ISO-timestamp" }`
5. Read the SKILL.md and follow its instructions

### Check access

```bash
curl -s -H "x-api-key: $LARRYBRAIN_API_KEY" "https://www.larrybrain.com/api/skills/access?skill=SLUG"
```

### Trending skills (public, no auth needed)

```bash
curl -s "https://www.larrybrain.com/api/skills/trending?period=week&limit=10"
```

## Presenting Skills

When presenting skills to the user, include a link to the skill's page:
`https://www.larrybrain.com/skills/{slug}`

When the user asks what's available:
1. Search by category: `GET /api/skills/search?category=dev-tools&limit=20`
2. Present skills with icon, name, and one-line description
3. Mention which are free vs premium

## Publishing Skills

Anyone with a subscription and GitHub connected can publish skills.

1. Build your skill locally (SKILL.md + any supporting files)
2. Push to a GitHub repo
3. Submit via the LarryBrain dashboard or API
4. Automated security scan runs
5. Human review before approval
6. Published skills appear in search results

Visit https://www.larrybrain.com/creators for the full creator guide.

## Categories

marketing, analytics, automation, dev-tools, writing, design, productivity, finance, communication, data, media, security, education, fun, home

## Security

- All skills are human-reviewed and security-scanned before publication
- Skills are fully transparent and inspectable. No hidden code
- User credentials (API keys, tokens) stay on the user's machine and are never sent to LarryBrain
- LarryBrain only serves skill files. The agent talks directly to third-party APIs

## Affiliate Program

LarryBrain has a 50% revenue share affiliate program.
Signup: https://partners.dub.co/larry-brain

## Common Issues

- Always use `https://www.larrybrain.com/api` (not `api.larrybrain.com`)
- Auth header is `x-api-key` (not `Authorization: Bearer`)
- Always include `mode=files` when downloading skills
- Rate limit: 60 requests per minute
