---
name: product-development
description: Find where a product should improve, by reviewing its features with the owner or digging through the services it connects to, then build each decided change as its own PR in parallel. Use when the owner wants to decide what to change across a product, where to focus next, or to resume that work. Use ce-ideate for ideas on one topic, ce-debug for one failure, and lfg for one change.
argument-hint: "review <area> | investigate [question] | resume"
---

# Product development

**Goal: density of work done.** The owner decides; many changes build at once while the conversation keeps going. Anything that can run at the same time does: hand independent work to helpers in one batch, and keep the main session for putting results together and talking to the owner.

This skill only does what Compound Engineering does not: the feature review, the digging through connected services, running many `lfg` at once, and merging by risk. Everything else is a CE skill, and CE skills write every record. This skill keeps no files of its own.

**Done:** the requested review or investigation is covered, and every change the owner chose is merged, waiting on the owner, or reported to the owner as stuck with what was tried.

## Find work

**Review** (`review <area>`). Split the area into parts and give each part to its own helper, all at once. Each helper reads its part's code at the latest main and, when services are confirmed, looks up read-only how many people used each thing in the last 90 days.

First show the owner the short list of parts and let them pick where to start. Then describe one part per message and stop for "next". Say what each thing does for a person, without file names, and add its usage ("used by 12 people in 90 days", "no use seen in 90 days", or "usage not checked"). Do not ask for a verdict on each item; the owner reacts to the lines they care about.

**Investigate** (`investigate [question]`):

1. **Services.** The first time, ask where the product runs and whether you may read its configuration. Read only the names of the connected services, never the values. The owner confirms the list. Save it the way `ce-product-pulse` does: which services the product uses (for example `posthog`, `intercom`, `postgres`) in the shared `.rocketclaw/config.yaml`, reusing the `pulse_*` keys where they exist and `product_` keys otherwise; how to reach each one (hosts, SSH names) in `.rocketclaw/config.local.yaml`. Never credentials or commands in either file. A change to the shared file goes in as a PR like any other change, merged by risk. Later runs check each one still answers with a read-only request and report new or missing ones.
2. **Existing reports.** The investigation's time window is the last 14 days, unless the owner's question names another period. Read the latest `ce-product-pulse` report and `ce-sweep` plan. If one is missing or does not cover that window, offer to run it for that window, and say that `ce-sweep` also acknowledges new items in Slack or GitHub. On yes, run it, decline its offer to schedule, and continue.
3. **Dig.** One helper per confirmed service, all at once, all on the investigation's time window. Each counts people or accounts where the service knows them. Where it does not, as with most logs, it counts events and labels them "people unknown"; never treat an event count as a count of people. Each finding says where to look again. Read-only only: a read-only database role or transaction, read-only commands over SSH. If only write access exists, the helper stops and reports it. Never copy secrets; never write names, emails, message text or account IDs to any file.
4. **Rank.** Join findings from different services that share a cause. Rank the problems by people affected and show the owner up to five, each with its evidence across services and how many people (or "people unknown"); "show me the rest" lists the others. Save the evidence, counts only, under `<repo-root>/.tmp/`. For each problem the owner picks, pass that evidence file to `ce-ideate` when the owner wants ideas, or go straight to Decide when they already know what to do.

## Decide

When the owner's words already settle what to do ("delete it", "make it optional"), that is the decision: go straight to Build. When the item is still open, start a separate child session for it, give the owner its link, and keep going here. The child runs `ce-brainstorm` with `mode:return-to-caller`, talks with the owner there one question at a time, and only decides: it does not build. Each time it finishes a round it reports back to this session (in OpenCode, create the session with this session as `parentID`); when its report holds a settled decision, take it to Build. Several brainstorms can be open at once. If the harness cannot create a child session the owner can talk in, run `ce-brainstorm` here instead. "Leave it" ends there, with no PR.

## Build

Each change goes to its own helper as soon as it is decided, and the conversation goes on. The helper runs `lfg` with the settled decision (the `ce-brainstorm` result, or the owner's own words), in its own workspace, and returns the PR link and the path of the plan `lfg` wrote, or why it stopped. Its brief says: if building raises a product question the decision does not answer, stop and send the question back; do not pick an answer. Bring that question to the owner like any other. One change, one PR. Start every independent helper at once; two changes to the same part of the product run one after the other, not at the same time. If two helpers fail with the same error, it is almost always the setup, not the changes: stop the other helpers, fix the setup once, then restart them, and tell the owner in one line.

## Merge by risk

When a PR's checks pass, start a separate reviewer, not the helper that built it, to score that commit with `merge-risk`, giving it the owner's decision and the plan `lfg` wrote. If it cannot be scored, treat it as medium.

- **Low:** if its head is still the scored commit, merge it the repo's usual way. If it goes into a merge queue, watch it until it lands or is bounced; if bounced, rebase, check, score again, and try again. Once it lands, get the landed commit from the PR's own merge record, never by searching the history, confirm it matches what was tested, and report one line.
- **Medium or high:** hold it for the owner, in one batch shown when no question is open or when the owner asks for progress: one line per PR with its risk, the reason, and what changes for a person. The owner answers "merge all", "merge 14", or "explain 15".
- **Always the owner, whatever the score:** permissions, auth, secrets, stored data or its schema, and anything that cannot be undone.

If a PR changes after scoring or approval, score and ask again.

## Owner gates

Running this skill allows, without asking: reading the repo and the confirmed services; helpers and workspaces; opening and pushing PRs; merging low-risk PRs as above. Everything else needs the owner's words:

| Action | Words needed |
|---|---|
| Merge a medium or high-risk PR | "merge <N>", "merge all", or "merge N after checks pass" |
| Close a PR | The owner says so |
| Read a deployment's configuration, or an unconfirmed service | The owner agrees |
| Run `ce-product-pulse` or `ce-sweep` | The owner says yes |
| Deploy to staging | The owner asks, or a standing preference covers it |
| Deploy to production | The owner asks, each time |
| Delete or migrate stored data, or change its schema | The owner chooses it after seeing the numbers |
| Message other people | Only when told, with the named identity |
| Other people's PRs | Never touch them |
| Add process beyond what this skill describes (locks, serial runs, extra gates) | The owner approves it first |

**The owner's rules.** When the owner states a general rule for this project ("match OpenCode V2 as closely as possible", "this has to work with CE"), save it word for word under `product_preferences` in `.rocketclaw/config.local.yaml`. Copy the saved rules into every helper's brief, and never write a stop into a brief that a rule already answers. Before asking the owner anything, reread the rules: if one answers the question, do not ask; apply it and say so in one line ("Following your 'match OpenCode', IDs now come from the folder"). Ask only when no rule covers it or two rules conflict.

## Talking to the owner

- Before you say anything, check it against the code at the latest main or the service you read. No caller in the repo does not prove a feature is unused if it can be reached from outside: say what you searched. Never raise a failure you cannot trace.
- One open question at a time. Background news gets one line, then the open question again.
- Answer a question from the owner in the first sentence. A question is not a decision.
- Write every message plainly from the start, like one person talking to another, with no jargon. Before sending, check: it starts with the answer; it has at most one question; every term is one the owner has used or is explained in plain words. When you raise a problem, give the fix in the same message.
- If the owner did not follow (`bro`, `dude`, "I'm lost"), say it again in fewer, simpler words, with an example, and save what confused them, in their words, under `product_preferences` so it does not happen again.
- Use the owner's words. Ask before adding something they never asked for that changes what they approved.
- "Progress?": what finished, what is running, what needs the owner. ETAs only from measured times. Name PRs by number and short name.
- When you wait, say which job will report back. If none can, say you will check by hand.

## Resume

Read the owner's standing preferences and the confirmed services from `.rocketclaw/config.yaml` and `.rocketclaw/config.local.yaml` (local wins key by key), then check live the open PRs by the owner's commit identity, the latest ideation file, and recent plans. If it is unclear which PRs belong to this work, list them and ask once. Find brainstorm sessions still open from an earlier session: read their results yourself, and give the owner their links to carry on there. Tell the owner what is waiting on them and what you will restart. For a review, show the list of parts again and ask where to continue. Do the same after a context compaction.

Scratch files go only under `<repo-root>/.tmp/`.
