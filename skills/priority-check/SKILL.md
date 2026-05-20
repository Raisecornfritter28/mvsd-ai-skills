---
name: priority-check
description: Grounds the user against their own stated priorities by rating the current chat or project as High, Medium, or Low priority. Built for people who lose hours to FOMO, ADHD-driven hyperfocus, escapism, or just having too much fun on side projects, and want a lightweight signal to stay aware of how they're spending their time and attention. Use this skill whenever the user invokes "priority-check", "priority check", "fomo-check", "fomo check", or asks "is this worth my time", and also when an instruction in user preferences directs Claude to run a priority check at the start of a chat. Supports four commands (chat, project, help, reset) and walks first-time users through a short questionnaire to set up their priority tiers.
---

# Priority Check

A grounding tool for people who lose hours to interesting-but-low-value pursuits. It reads the user's defined priority tiers (or helps them define some) and rates the current conversation or project against those tiers, prepended as a single line so it's a reminder, not an interruption.

Built for the kind of person who knows they over-invest in side projects, rabbit holes, or fun-but-pointless work, and wants a lightweight signal -- not a blocker -- to stay grounded. Especially useful for users with ADHD, FOMO-driven attention drift, executive function challenges, or anyone who wants their AI assistant to surface trade-offs rather than enable drift.

The skill is fully self-contained. On first use, it walks the user through a short questionnaire to establish their priority tiers, then generates a paste-ready settings block for their Claude preferences. On subsequent uses, it reads those settings and rates the topic at hand.

---

## Commands

The skill accepts one of four commands as part of its invocation. If no command is given, default to `chat`.

| Command | Behavior |
|---------|----------|
| `chat` (default) | Rate the current conversation against the user's priority tiers |
| `project` | Rate the current project (if the user is in one) against the priority tiers |
| `help` | Show what the skill does and list the user's current priority settings |
| `reset` | Re-run the questionnaire to redefine priority tiers (asks for confirmation first) |

Examples of how the user may invoke:
- "priority-check" → run `chat`
- "priority-check project" → run `project`
- "priority check help" → run `help`
- "fomo check" → run `chat` (alias)
- "run a priority check" → run `chat`

---

## Settings block format

The skill's configuration lives in the user's preferences (or a project instructions file) inside a clearly delimited block:

```
=====
%%PRIORITY CHECK SETTINGS v1%%
high: <comma-separated topics>
medium: <comma-separated topics>
low: <comma-separated topics>
run_on_chat_start: <true | false>
At the start of any substantive new conversation, invoke the priority-check skill to rate the topic.
=====
```

The five equals signs and `%%PRIORITY CHECK SETTINGS v1%%` header are exact -- they're how the skill finds and updates its own block. The version marker (`v1`) lets future versions detect and migrate old formats.

The trailing instruction line is what tells Claude to actually invoke the skill on chat start. It's only included when `run_on_chat_start` is `true`. When the flag is `false`, omit that line from the block.

Any tier can be empty if the user didn't specify topics for it. Only `high` is required to be non-empty after the questionnaire. Empty tiers are written as `medium:` or `low:` with nothing after the colon.

When generating or regenerating this block, always use this exact shape.

---

## Tier inference rules

The user is only required to define High priority topics. Medium and Low are optional. For any topic encountered during a check that doesn't exactly or near-exactly match a listed tier, infer the tier using these rules in order:

1. **Universal-importance topics default to High** even if not configured: legal matters, financial decisions of consequence, medical or health issues, family or relationship crises, safety matters, parenting decisions. Rate these High and note "universal-importance topic" or similar as the reason.

2. **Topics related or semi-related to the user's High list are Medium.** "Related" means same domain, adjacent skill area, supporting infrastructure, or topics the user would plausibly need to engage with to succeed in their High areas.

3. **Topics unrelated to any High topic and not universally important are Low.**

If the user has explicitly listed topics under Medium or Low, an exact or near-exact match to those lists overrides inference. Otherwise, infer.

---

## Workflow

### Step 1: Detect the settings block

Look for the `%%PRIORITY CHECK SETTINGS%%` marker in the user's instructions, preferences, or project context.

- **If found:** parse the tiers and the `run_on_chat_start` flag. Proceed to the requested command.
- **If not found:** any command other than `help` should first run the questionnaire (Step 2). After the questionnaire completes, continue to the requested command.

### Step 2: First-time questionnaire

Only run if no settings block exists, or if the user invoked `reset`.

For `reset`, first ask: "This will replace your existing priority check settings. Continue?" Wait for explicit confirmation before proceeding.

**For first-time users only** (no prior settings block ever found), open with a brief intro before the first question:

> Welcome. Priority Check rates each conversation against priorities you define, then prepends a single-line tag to my responses so you can stay grounded on where your attention is going. It's a reminder, not a blocker -- you decide what to do with it.
>
> I'll ask a few short questions to set this up. Takes about a minute.

For `reset` re-runs, skip the intro -- the user already knows what the skill does.

Then ask, one at a time:

1. "What topics or areas of your life are **High** priority?"
2. "Are there topics you consider **Medium** priority? You can list them, or just say 'no' and I'll infer based on how related a topic is to your High list."
3. "Are there topics you consider **Low** priority -- things you tend to over-invest in and regret? You can list them, or say 'no' and I'll infer."
4. "Do you want the priority check to run automatically at the start of every substantive new chat? (yes/no)"

Keep questions terse. Don't add framing or examples unless the user asks. Trust the user to know what these words mean for them.

If the user answers "no" to questions 2 or 3, leave that tier empty in the settings block. The inference rules will handle topic classification at check time.

After collecting answers, generate the settings block (in the format above) and present it to the user with paste-ready instructions:

> Here's your priority check settings block. Add this to your Claude preferences (Settings → Profile → Preferences) so the skill can find it on future runs:
>
> ```
> =====
> %%PRIORITY CHECK SETTINGS v1%%
> high: ...
> medium: ...
> low: ...
> run_on_chat_start: ...
> At the start of any substantive new conversation, invoke the priority-check skill to rate the topic.
> =====
> ```
>
> If `run_on_chat_start` is `false`, omit the trailing instruction line.

Once the user confirms they've added the block, continue to the originally requested command.

### Step 3: Run the requested command

#### `chat` (default)

Assess the current conversation's topic against the user's tiers. Output a single line, prepended to your normal response:

```
Priority: [High | Medium | Low] -- [one short reason]
```

Then proceed with the actual response to whatever the user asked. The priority line is a grounding signal, not the response itself.

If the topic genuinely doesn't fit any defined tier, see "Ambiguous topic handling" below.

#### `project`

Same as `chat`, but assess the *project* the user is in (if any), based on its title, instructions, and visible scope. Projects represent sustained investment, so the output is slightly richer:

```
Project priority: [High | Medium | Low] -- [one short reason]
Trade-off note: [one sentence on what this project might be displacing, if Low or Medium]
```

If the user is not in a project, say so and offer to run `chat` instead.

#### `help`

Output a verbose explanation:

> **Priority Check** rates your current chat or project against priority tiers you've defined, so you can stay aware of how you're spending your attention.
>
> **Your current settings:**
> - **High:** [list]
> - **Medium:** [list or "inferred from High"]
> - **Low:** [list or "inferred (anything unrelated to High)"]
> - **Run on chat start:** [true/false]
>
> **Commands:**
> - `priority-check` or `priority-check chat` -- rate this conversation
> - `priority-check project` -- rate the current project
> - `priority-check help` -- show this message
> - `priority-check reset` -- redefine your tiers
>
> The check is a grounding signal, not a blocker. It surfaces the trade-off; you decide.

If no settings block exists yet, say so and offer to run the questionnaire.

#### `reset`

Confirm with the user, then re-run the questionnaire (Step 2). Generate a new settings block and ask the user to replace the old one in their preferences.

---

## Ambiguous topic handling

In most cases, the inference rules above will produce a confident tier without needing to ask. Only fall back to asking the user when the topic genuinely doesn't fit any defined tier *and* inference produces a low-confidence result (e.g., the topic is novel, multi-domain, or borderline universal-importance).

When that happens:

1. Briefly summarize the topic in one sentence.
2. Ask: "This doesn't clearly fit your tiers and I'm not confident inferring it. Is this **High**, **Medium**, or **Low** priority?"
3. After the user answers, output the priority line as normal.
4. Then ask: "Want me to add this topic to your **[tier]** list so it's recognized next time? I'll generate an updated settings block for you to paste."
5. If yes, regenerate the full settings block with the new topic added to the appropriate tier and present it paste-ready.

Don't push. If the user says no, drop it and continue.

---

## Output style

- The priority line is **always one line**, prepended to the response.
- Reasons should be one short clause -- not a sentence with multiple clauses.
- No moralizing, no encouragement to switch tasks, no commentary on whether the user "should" do the thing.
- If the topic is High, the reason should affirm fit briefly (e.g., "core career area").
- If the topic is Low, the reason should name the tier match plainly (e.g., "creative side project") without judgment.

Examples:

```
Priority: High -- Salesforce architecture, core career track.
```

```
Priority: Low -- creative worldbuilding, classified as low-ROI side project.
```

```
Priority: Medium -- DIY, fits home management tier.
```

---

## Edge cases

**Trivial chats.** If the user opens with a one-shot trivial query ("what's 2+2", "translate this word"), don't run the check even if `run_on_chat_start` is true. The check is for substantive engagement.

**Multiple topics in one chat.** Rate the dominant or initiating topic. Don't try to rate every sub-thread.

**User pushback.** If the user says the rating is wrong, accept the correction without arguing. Optionally offer to update their tiers via `reset` or by adding the topic to a tier.

**Settings block exists but is malformed.** Tell the user the block looks corrupted, show what was found, and offer to regenerate it via `reset`.
