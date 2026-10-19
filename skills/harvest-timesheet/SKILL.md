---
name: harvest-timesheet
description: Turn a day's work into Harvest time entries, from the daily-update sources the user has configured or from what they tell you, then submit through the Harvest MCP server or hand over a draft. Use when the user asks to end the day, fill in or submit their timesheet, or log today's time in Harvest.
---

# Harvest Timesheet

Turn one day's work into Harvest time entries. Work is gathered from the user's **sources** (the places they keep daily updates, named in their config), then the user maps every item to a Harvest project and task and supplies the hours. Harvest tools come from the `harvest` MCP server and appear as `mcp__harvest__<tool>` in Claude Code and `mcp__harvest.<tool>` in Codex.

The user owns every choice that ends up in Harvest: which items become time, the project and task for each, and the hours. The skill gathers, organizes, and drafts; suggestions come only from the user's saved mappings.

## Config

The config lives at `~/.cache/harvest/timesheet.md`. It lists the user's sources and any saved project mappings. When it is missing, or the user wants to change where updates come from, follow [CONFIG.md](CONFIG.md) to set it up, then continue.

## Steps

1. **Set the date and clock.** Default to today's local date, from the environment. Also read the current local time (run `date '+%H:%M'` when the environment does not give it). Use the exact `YYYY-MM-DD` date in every prompt and in the draft.
2. **Gather the day's work.** Read every source in the config for that date. Then ask the user what else they worked on that the sources don't show (meetings, calls, email, reviews). When the user is the only source, ask them to describe the day in their own words; a rough list is enough. This step is done when every source has been read and the user has had a chance to add more.
3. **Check what's already logged.** Call `list_time_entries` with `from` and `to` set to the date. Show those entries, so the draft covers only what's missing.
4. **List the candidates.** Turn the work into one numbered list. Each item gets a short summary in plain words, its source, and its status (`done`, `in progress`, or `meeting`), plus the time range when a source gives one. Merge duplicates that several sources report. Ask which items to combine, skip, or carry forward with no time.
5. **Map each item.** Read the project cache written by `refresh-harvest-projects` (its path is in that skill). If the cache is missing or has `needs_refresh: true`, say so and offer to run `refresh-harvest-projects`, or continue with project and task names the user types in. Show the projects by `display_name`, and the chosen project's `task_names` as its tasks. Pre-fill items from the config's saved mappings, marked as suggestions. Ask the user for the project and task of every remaining item; grouped answers work ("1 and 3 go to Acme: Website / Development"). When the user names a project or task the cache lacks, say so and ask whether to use their value.
6. **Get the hours.** Ask for hours per item or per project-and-task group. A meeting's time range is a reference for the user to confirm. Present a day total that looks low or high plainly and ask about it.
   - **Size the day from the workday start in the config.** Read the user's workday start time from the config's Preferences (default 9:00am local). The hours to account for up to the moment the skill runs are `current local time − workday start`, as decimal hours. A run at 1:30pm with a 9:00am start is 4.5h so far. Use that as the running total for a run before the end of the day; by day's end it reaches the user's daily target.
7. **Draft.** Show the draft table (below) and ask the user to approve or correct it. This step is done when every candidate is mapped, combined, carried forward, or skipped, and every entry has hours the user gave.
8. **Finish one of two ways**, as the user chooses:
   - **Submit to Harvest.** Resolve each entry's project to its `project_id` from the cache's `projects` and its task to `task_id` from `task_assignments`. When an ID can't be resolved, ask the user how to proceed. Restate the exact entries and get an explicit yes, then call `log_time` once per entry with `project_id`, `task_id`, `hours`, `spent_at` (the date), and `notes`. Report each entry that was logged and any that failed.
   - **Hand over the draft.** Give the final table, plus one copy-ready line per entry (`Project / Task: hours: notes`), for the user to enter themselves.
9. **Save new mappings.** When the user mapped a recurring label (a project heading, a story-ID prefix, a calendar name) during this run, offer to add it to the config's mappings for next time.

## Draft table

| Date | Source | Work item | Harvest project | Harvest task | Hours | Notes |
| --- | --- | --- | --- | --- | ---: | --- |

Notes are short, past tense, and fit for a timesheet: what was worked on, with the story IDs and deliverable names from the sources. In-progress work reads as "worked on" or "continued"; "completed" appears only when a source says the work finished.
