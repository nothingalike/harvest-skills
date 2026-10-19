# Timesheet config

How `harvest-timesheet` learns where a user keeps their daily updates. Read this when the config is missing or the user wants to change it.

## The file

`~/.cache/harvest/timesheet.md` is plain Markdown. Sources say, in plain words, where a day's work can be found and how to read it; `{date}` stands for the work date as `YYYY-MM-DD`. Mappings are labels the user has already tied to a Harvest project and task. Preferences is optional and holds settings like when the workday starts.

```markdown
# Timesheet config

## Sources
- Daily notes: `~/notes/daily/{date}.md`. Every bullet under "Worked on" is a work item.
- Ask me: after reading the files, ask what else happened today.

## Mappings
- `acme-shop` → Acme: Website Rebuild / Development
- Story IDs starting `BILL-` → Billing Co: API Support / Development

## Preferences
- Workday starts at 9:00am local.
```

A workday-start line in Preferences (`Workday starts at 9:00am local`) lets the skill size a run before the end of the day: the hours to account for so far are the current local time minus the start. With no line, the skill defaults to 9:00am.

A source can be anything the agent can read:

- a file or folder of notes, with the path pattern and which headings or markers count as work;
- a git repository, read for the day's commits by the user (`git log --since=... --author=...`);
- a calendar or task tool the agent has an MCP connection to, naming the server;
- **Ask me**: the user describes the day in chat. This is a complete config by itself.

Write each source so a fresh agent could read it with nothing else to go on: where it is, what marks a work item, and which project a line belongs to when the source says.

## Setup

1. Ask the user how they keep track of their day. Offer the kinds above as examples, and make clear that "I'll just tell you" is a fine answer.
2. For each place they name, look at one recent day's entry together, to confirm the path pattern and what marks a work item.
3. Write the config with their sources, and an empty Mappings section unless they already know some.
4. Show the file, and tell them they can edit it by hand or ask to change it any time.

## Example: headquarters plate

A user of the `plate` skill from headquarters keeps the day's log in `~/.headquarters/plate`:

```markdown
## Sources
- Plate log: `~/.headquarters/plate/log/{date}.md`. Every line under Done, Blockers, Questions, and Notes is evidence of the day's work. Lines start with their plate project (`acme-shop: ...`); a line without one is placed by its story ID, or asked about.
- Plate: `~/.headquarters/plate/plate.md`. Items marked `[/]` are in progress and may have been worked on today, so list them as `in progress` candidates for the user to keep or skip.
- Ask me: after reading the plate, ask what else happened today.
```
