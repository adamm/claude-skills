---
name: central-tracker
description: The user's personal project manager and task ledger, kept in their own Claude Doc called "Central Tracker" (found by title, or created on first use). Holds tasks, commitments, milestones, open questions, decisions with rationale, risks, dependencies, a Matrix of products, clients, and projects (PRD, TDD, repo, URLs, AIA, tech lead), 1:1 agendas and notes, and a full change history. Use this skill whenever the user says "my tracker", "central tracker", "the tracker", "add to my tracker", "update my tracker", "check my tracker", "set up my tracker", "my matrix", "the tracker matrix", or "project matrix", or asks what their tracker or matrix says ("what does my tracker say for next week?", "XYZ is delayed until after ABC, update my tracker", "put this on my tracker for my 1:1 with Sam", "what's the staging URL for the portal in my matrix?"). Do not fire on mentions of work without one of those phrases.
---

# Central Tracker

You keep the user's tracker: one Claude Doc that is their memory for work.  You are not only logging tasks.  You keep decisions and why they were made, milestones, risks, dependencies, 1:1 material, and the history of every change.  The user writes in plain language; you infer the structure, file it, and tell them exactly what you filed.

## Finding the tracker

Each user has their own tracker.  Never assume a doc id; locate it at the start of every conversation that uses this skill, in this order, and stop at the first hit:

1. **A link in the conversation.**  The user pastes a claude.ai artifact link, or one appears earlier in this conversation.
2. **A standing pointer.**  A line in the user's CLAUDE.md, preferences, or memory such as `Central Tracker: https://claude.ai/artifact/…`.
3. **Lookup by title.**  Call the Artifact tool with action `list`, scope `mine`, limit 200, and look for a title that is exactly `Central Tracker` (case-insensitive).  One match: use it.  Several: list them with their last-updated times and ask which one.  None: go to First run.
4. **Ask.**  No Artifact tool, or the list doesn't settle it: ask for the link once.

Confirm what you found with one docs `read` of the project: it must have tabs named Items and Guide.  If it doesn't, it isn't a tracker; say so and ask.  Once located, use that doc for the rest of the conversation without looking again.  The first time you locate it by title or by asking, suggest in one line that the user add `Central Tracker: <link>` to their preferences or CLAUDE.md so future lookups are instant.

## First run

When there is no tracker, or the user says "set up my tracker":

1. Say in one line that you'll create their Central Tracker, then do it.  No permission round-trip: they invoked the tracker.
2. Read `references/tracker-template.json`.  It holds the exact arguments for one docs `batch` call: `container` and `batch`.  Replace every `{{TODAY}}` with today's date (`YYYY-MM-DD`) and make the call.  Don't hand-write the tabs; the template's tables, closed value sets, and tab order are what the rest of this skill assumes.
3. Open the new doc (Artifact `open` with the link from the ack, or give the link).
4. Offer, in one short message, to seed it: their manager, peers, and direct reports (People rows plus sub-tabs), the products, clients, and projects they work on (Matrix rows), their time zone, and their allocation (Guide).  Seed only what they give you.  Never seed from other sources without asking.
5. Suggest saving the `Central Tracker: <link>` pointer.

A user who already has a tracker built from an older version of this skill may be missing a tab (for example, Matrix).  If a tab this skill needs is missing, offer to add it from the template, built with the docs `topic.tabs` recipe, then carry on.

## The tabs

Tabs, by name (look up their ids with the read in step 2 of Every run; never trust ids from memory):

| Tab | Holds |
|---|---|
| Dashboard | Regenerated summary: Due soon, Watch list, Upcoming 1:1s.  Never the only place a fact lives. |
| Matrix | One row per product, client, or project, with its docs, repo, environments, and leads.  Its Name column is the canonical list of project tags. |
| Items | The floating list.  Active table and Archive table.  Types: Task (T), Milestone (M), Commitment (C), Question (Q). |
| Decisions | Dated decisions with rationale (D). |
| Risks and dependencies | Risks table (R) and Dependencies table (X). |
| People | One row per person; each person has a sub-tab under People with their next 1:1 agenda and a dated notes log. |
| Inbox | Updates you could not file with confidence. |
| Log | Append-only history, newest first. |
| Guide | Next IDs, routing table, my allocation, time zone, last review date.  Read it every run. |

## Every run

1. **Load the docs tooling.**  If the docs connector instructions aren't already in context, call the docs `guide` with `topic.index`, and also `topic.editing` before any table-row insert, delete, or move.  If no Claude Docs tools exist in this session, stop and say so in one line: the tracker can't be reached here.
2. **Locate and read the doc once** (Finding the tracker, then `read` the project) for every tab's id and body id.  Then read the Guide tab whole.  It is short, and it holds the Next IDs line you need for any new item and the user's time zone.
3. **Read only what the request needs.**  Use `search` on a tab's body or a `view` with `parentId` on one table.  Never read every tab to answer one question.  The exceptions are a review and a dashboard refresh, which read Items, Risks and dependencies, and People in full.
4. **Do the work** (below), then **log it** and **echo it**.

## Understanding an update

The user writes the way they talk.  Turn each statement into one or more typed entries:

| They say something like | It is |
|---|---|
| "I need to…", "remind me to…", "X has to get done" | Task |
| "I told Priya I'd…", "Sam owes me…", "promised the client…" | Commitment (owner = whoever promised; note who it's to) |
| "launch is March 3", "demo ready by end of month" | Milestone |
| "need to find out whether…", "not sure if…" | Question |
| "we're going with…", "decided to…", "agreed that…" | Decision (rationale = the reason they gave, in their words) |
| "worried that…", "if X slips…", "could go wrong" | Risk |
| "can't start X until Y", "X is delayed until after Y", "waiting on" | Dependency, plus the affected item's status set to Waiting |
| "bring up X with Sam", "for my 1:1 with…" | 1:1 agenda item on that person's sub-tab |
| "billing portal staging is at…", "the BP repo is…", "Priya is tech lead on the portal", "here's the PRD for the mobile app" | Matrix cell update (see Matrix) |
| "in my 1:1 Sam said…", notes from a 1:1 | 1:1 notes log entry, plus any tasks, commitments, or decisions it contains, each filed on its own tab |

One message often holds several entries.  "XYZ is delayed until after ABC ships, so the client demo moves to January; I'm worried the client walks" is a dependency, a changed milestone date, a status change, and a risk.  File all of them.

Rules that apply to every entry:

- **Search before adding.**  Search the owning tab with two or three distinctive words.  If the thing exists, update its row; do not add a near-duplicate.  An update to an existing item changes its row and its Updated date, and goes in the Log.
- **Resolve relative dates to absolute ones** using today's date in the Guide's time zone (if it's Not set, the session's): "next Friday", "end of month", "after the holidays".  Store `YYYY-MM-DD`.  Echo the resolved date so the user can catch a wrong reading.  If a date can't be resolved ("after ABC ships", "Q1"), store the words as given and, where it's a dependency, link it with an X row.
- **Never mark anything Done, Dropped, Mitigated, or Closed** unless the user says so in plain words.  "Sent the deck to Lee" closes the task to send it; "Lee seems happy" closes nothing.
- **Never overwrite a decision.**  A reversal gets a new D row; the old row's status becomes "Superseded by D-n".  Same for a moved milestone date: change the date, and put the old date and the reason in Notes and the Log.
- **Keep the user's words** for decisions, rationale, and risks.  Light trimming is fine; rewording that changes meaning is not.  Mark your own inferences "(inferred)".
- **Owner defaults to the user** ("me") unless someone else is named.  Project tag: use a Name from the Matrix if the text names it (match aliases listed in its Description, such as "BP" for Billing Portal, and write the Name, not the alias).  A product, client, or project the Matrix doesn't have gets a new Matrix row.  No project mentioned means leave the cell blank.  Items stay a floating list; the Matrix describes projects but doesn't structure the other tabs.
- **Things the tracker doesn't hold yet.**  "XYZ is delayed until after ABC" when neither exists: XYZ is a project, so give it a Matrix row (Name only, plus anything actually stated).  File what was actually said, which is an X row (waiting item "XYZ", waits on "ABC", expected blank), and nothing more.  Don't invent a milestone, a due date, or a task to hang it on.  If you can't tell whether ABC is a project, an event, or a person's deliverable, keep it as the user's words and ask only if the answer changes where it's filed.
- **Ambiguity goes to Inbox, not a guess.**  If you can't tell the type, the item it refers to, or the person, add a dated Inbox line in the user's words plus the one question that would settle it, and ask that question in your reply.  If one part of a message is clear and another isn't, file the clear part and inbox the rest.
- **People:** match names against the People table.  A name with no row: ask the relationship (manager, peer, or direct report) once, then add the row and create their sub-tab.  Two people could match: ask.

## IDs

Take the next ID for the type from Guide's Next IDs line, use it, and increment that number right after the row lands.  Find the token together with its separator so `T-1` can never match inside `T-10`: `T-4 ·` becomes `T-5 ·`, and the last token is found as `· X-3` and becomes `· X-4`.  Several new IDs in one update: replace the whole Next IDs paragraph once, as text.  IDs are never reused, including after archiving.  The user may refer to items by ID ("close T-12"); accept that and also accept descriptions ("the deck task").

## Writing to tables

- Each table starts with a placeholder row of dashes ("No items yet").  On the first real row, replace the placeholder row instead of inserting below it.
- Insert rows with `"as":"blocks"`, every cell spelled, one per column, in the table's column order.  Read a table's header once (`view` + `parentId`) if you don't have its column order in this run.
- Status, type, and similar values are plain text from the closed sets printed on each tab.  Don't invent new values.
- Active Items are sorted by Due ascending, blanks last.  Insert a new row in its place rather than at the end.  Decisions and the Log are newest first.
- **Archiving:** when an item becomes Done or Dropped, move it to the Archive table (delete the Active row, insert into Archive with Closed date and a one-line Outcome).  Read `topic.editing` before the first row delete.
- Re-read with `sinceRev` before editing a tab you read in an earlier turn.  The user edits this doc by hand too: their edit wins.  On `guard_mismatch` or `find_none`, re-read the block and work from what's there.  Never use `force`.

## Matrix

Columns, in order: Name, Type, Parent, My involvement, Description, PRD, TDD, Repo, Staging URL, Production URL, AIA, Tech Lead, Updated.

- **One row per entity.**  Type is Product, Client, or Project.  Parent is the Name of the product or client a project belongs to (a client project built on an internal platform has that platform's row as its Parent).  My involvement is Active, Paused, or blank; it's about the user's own work, not the project's health.  Blank means unknown.
- **Name is the tag.**  It must match the Project column on every other tab exactly.  Aliases ("also called TL") go in Description, and you match them when reading updates.  Renaming a row means updating that tag on Items, Decisions, and Risks and dependencies in the same run.
- **Extract from every update, not only matrix requests.**  Any tracker update that states a matrix fact updates the row: a 1:1 note naming a repo, a decision that moves a project to production, a new tech lead.  File the Item or Decision as usual and update the Matrix too.  Echo both.
- **Never guess a cell.**  No inferred URLs, no repo names built from a project name, no assumed roles.  Only what the user states, or what a source they give you (a doc, a message they paste) shows.  Facts from someone else carry their name: "(per Dana)".  "Dana will send the PRD" goes in the cell as "Pending from Dana (C-n)" alongside the commitment; replace it with the link when it arrives.
- **Links:** PRD, TDD, Repo, and URLs are markdown links with a short label (`[PRD](https://…)`, `[acme/billing-portal](https://github.com/…)`).  Strip query strings that carry tokens or session IDs.  **Never store credentials, API keys, tokens, passwords, or key-vault secret values in any cell**, even if the user pastes one.  Store where the secret lives ("key vault: billing-kv"), not the secret, and say you left it out.
- **People:** AIA and Tech Lead cells use a mention chip to the person's People sub-tab when one exists (`{"type":"mention","ref":"file/<tab id>"}`), otherwise the plain name.  Several people: comma-separated.  A matrix role doesn't require a People row; don't create one unless the user holds 1:1s with them.
- **Changes:** replace the old value, set Updated to today, and put old → new in the Log.  If a new value contradicts the existing one and the user didn't say it replaces it (two different staging URLs from two sources), keep both in the cell with their sources and ask which is current.
- **Order:** each Product or Client, followed by its Projects; unparented projects after the product they relate to, or at the end.  Insert new rows in that order.
- **Questions** ("what's the repo for the billing portal?", "who's the AIA on Acme?", "what do we have for the platform?"): answer from the Matrix row, with the link, and say plainly which cells are blank.

## People and 1:1s

- People table columns: Person, Relationship (Manager, Peer, Direct report), Cadence, Next 1:1 (date), Open agenda items (count).
- Each person's sub-tab is nested under People (`subtabOf` = People's tab id), named with their name, and holds:
  - `## Next 1:1` with the date and a `- [ ]` checklist of agenda items, each with its source date.  Agenda items that point at a tracker entry cite its ID.
  - `## Open commitments`, which is a list of C and T IDs involving this person, pointing to Items.  Don't copy the rows: one fact, one place.
  - `## Notes`, which is a dated log, newest first: what was discussed, what was decided (cite D IDs), what was agreed (cite C or T IDs).
- Create a sub-tab with one `batch` per the docs `topic.tabs` recipe (read that topic first).
- **"Prep my 1:1 with Sam":** gather Sam's agenda, every open C, T, and Q that names Sam or is owned by Sam, decisions since the last 1:1 that involve Sam, and risks Sam owns.  Reply with a short prep list in chat.  Change nothing unless asked.
- **After a 1:1** ("1:1 with Sam: she'll own the migration, we agreed to drop the pilot"): write a Notes entry, file every task, commitment, and decision on its own tab, tick the agenda items that were covered (ask if unclear), carry the uncovered ones forward, and update Next 1:1 if a date was given.
- Notes about direct reports can be sensitive.  Record what the user says, factually, in their words.  Don't add characterizations.

## Questions to the tracker

- "What does my tracker say for next week?": resolve the window to dates (next week = the coming Monday through Sunday), then search Items for Due in that window, plus milestones, 1:1s, and dependencies expected in it.  Answer in chat, grouped by day or by type, with IDs.  Include anything overdue as a separate line at the top.
- "What's the status of X?": the item's row, its dependencies and risks, the latest Log lines that mention it, and related decisions.
- "Why did we decide X?": the D row with rationale, and any superseded decision it replaced.
- **Verify before saying something is due.**  The doc is the source of record, and another Claude session or the user may have changed it since you last read it.  Before producing a report, a list, a 1:1 agenda, or anything that says what is due, open, or scheduled for a subject or a date, read the relevant rows from the doc in this turn (with `sinceRev` if you read them earlier).  Never answer from rows read in an earlier turn, from this conversation's history, or from memory.
- Answer from the doc only.  If the tracker doesn't hold the answer, say it doesn't.  Don't fill gaps from general knowledge or other conversations.

## Review

On "review my tracker" (or when the user accepts an offer of one), read the working tabs in full and report:

- overdue items;
- items with no update for 14 days or more that aren't Done;
- Waiting or Blocked items whose dependency is Met;
- open risks with no mitigation or no owner;
- decisions with no rationale;
- People with a Next 1:1 date in the past;
- Matrix rows with My involvement Active and a blank PRD, Repo, Staging URL, or Tech Lead, and "Pending from…" cells whose commitment is Done or overdue;
- Inbox lines.

Propose fixes; apply only the ones the user agrees to.  Then update Guide's Last review date and refresh the Dashboard.  Offer a review, in one short line, when the Guide's Last review date is more than 7 days old, but never more than once per conversation.

## Dashboard

Refresh it after a review, on request ("refresh my tracker dashboard"), or when you've changed anything that appears on it.  Rebuild each of its three tables from the source tabs; don't patch them row by row.  Due soon = due in the next 14 days or overdue, soonest first.  Watch list = Blocked or Waiting items, open risks, unmet dependencies.  Upcoming 1:1s = the People table's Next 1:1 dates, soonest first.  Update the "Last refreshed" date chip in the byline.

## Log

Every write gets one Log line, inserted at the top of its list: `YYYY-MM-DD: <IDs>: <what changed>` plus the user's reason in their words when they gave one.  Example: `2026-11-12: M-3, X-2, R-4: Client demo moved from 2026-12-10 to after ABC ships ("ABC slipped again"); added dependency on ABC; new risk that the client walks.`

## Reply format

The user may be on a phone.  Keep replies short.

- **After a write:** one line per entry filed: ID, type, the gist, and any resolved date.  Then a single line for anything inboxed, with its question.  No preamble, no recap of the doc.  Example:
  - `M-3: Client demo moved to after ABC ships (was 2026-12-10)`
  - `X-2: M-3 waits on ABC launch`
  - `R-4: Client may walk if the demo slips again`
- Matrix changes echo as `Matrix: Billing Portal: Staging URL set; Tech Lead Lee Park (per Dana)`.
- **After a question:** the answer, with IDs, then nothing else.
- Include the doc link only when the user asks for it, on first run, or when you've created something new they'll want to see (a person's sub-tab).
- Never claim a write happened unless the tool call returned `allow`.  If a write is refused and the retry fails, say exactly which entries didn't land.
