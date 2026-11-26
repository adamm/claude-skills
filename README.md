# Claude Skills for Adam

My growing collection of skills build under Claude for general usage

## Central Tracker

A Claude plugin that turns one Claude Doc into your personal project manager and task ledger: tasks, commitments, milestones, open questions, decisions with rationale, risks, dependencies, a product and project Matrix (docs, repo, environments, leads), 1:1 agendas and notes, and a full change log.  You write updates in plain language; Claude files them and tells you exactly what it filed.

Each user gets their own tracker.  Nothing in this repo points at anyone's data.

### Install

**Claude Code**

```
/plugin marketplace add https://github.com/adamm/claude-skills.git
/plugin install central-tracker@adamm-claude-skills
```

Then say "set up my tracker." It creates your own private tracker doc.

Turn on auto-update for the marketplace in `/plugin` → Marketplaces, or refresh by hand with `/plugin marketplace update adamm-claude-skills`.

**claude.ai, Claude Desktop, and Cowork (organization-wide)**

An organization Owner connects this repo in Organization settings → Plugins → Marketplaces → Add plugins → Sync from GitHub.  The repo must be private or internal, and the Claude GitHub App must be installed on it (with the Webhooks permission approved, for automatic sync).  The Owner then chooses who gets the plugin: auto-installed, available to install, or hidden, per group.

### First use

Say "set up my tracker".  Claude creates a doc titled **Central Tracker** in your account, opens it, and offers to seed your people, projects, and time zone.  After that, any message with "my tracker", "central tracker", "the tracker", or "my matrix" uses it:

- "Add to my tracker: send the Q1 plan to Dana by Friday."
- "What does my tracker say for next week?"
- "1:1 with Sam: she'll own the migration, we agreed to drop the pilot.  Update my tracker."
- "The billing portal staging URL is https://…, update my matrix."

Claude finds your tracker by its title.  To make lookups instant, add this line to your Claude preferences (claude.ai) or `~/.claude/CLAUDE.md` (Claude Code):

```
Central Tracker: https://claude.ai/artifact/<your-doc-id>
```

Requirements: the Claude Docs connector.  Without it, the skill says the tracker can't be reached.

Your tracker is private until you share it.  It can hold 1:1 notes about the people you manage, so share exports of specific tabs, not the doc.

### Releasing changes

Edit `plugins/central-tracker/skills/central-tracker/SKILL.md` or the template, bump `version` in `plugins/central-tracker/.claude-plugin/plugin.json`, open a PR, merge to the default branch.

Claude Code caches the plugin by `version`.  If you merge a change without bumping it, users keep the cached copy.  Use semver: patch for wording fixes, minor for new behaviour, major for changes that need users to migrate their tracker.

Organization sync: if a sync fails, the plugin can be temporarily removed for everyone until a fixed commit is synced.  Validate before merging (`claude plugin validate .` from the repo root).

### Layout

```
.claude-plugin/marketplace.json                 marketplace catalog
plugins/central-tracker/
  .claude-plugin/plugin.json                    plugin manifest
  skills/central-tracker/SKILL.md               the skill
  skills/central-tracker/references/
    tracker-template.json                       one docs batch call that creates a new tracker
```
