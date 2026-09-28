# Home Maintenance Builder — a reusable Claude skill

**Home Maintenance Builder** is a Claude skill that turns a street address into a personalized home-maintenance plan. It researches the property from free public data, confirms the key facts with the homeowner, fits a seasonal schedule to the local climate, looks up local DIY and professional prices, and packages everything into a new, house-specific skill plus an interactive maintenance calendar and a set of monthly calendar reminders.

The research process was developed step by step in a Claude chat, then saved as a skill so it can be rerun for any house with one short prompt, such as *"Build a maintenance plan for 123 Main St, Springfield, IL"* or just an address and *"do this for this one."*

Built for the Week 6 assignment ("Let's Build a Research Skill") in the Foundations of AI course.

## What it accomplishes

Given an address, the skill produces three things:

| Output | What it is |
|---|---|
| `home-maintenance-<house>.skill` | A new skill for that one house. It knows the home's systems, climate, and local prices, and can answer questions like "What should I do this month?" or "My AC is dripping, what do I check?" |
| `maintenance-calendar-<house>.html` | An interactive 12-month calendar with checkable tasks, category filters, DIY-vs-pro cost toggles, and annual cost totals |
| `home-maintenance-reminders-<house>.ics` | Twelve yearly-repeating reminders (one on the 1st of each month) listing that month's tasks and costs; import once into Google, Outlook, or Apple Calendar |

It also reports the estimated **annual maintenance cost** two ways: doing everything DIY where reasonable, and hiring everything out.

## How it works

The skill follows four steps (defined in `SKILL.md`):

1. **Research the property.** Searches listing sites (Redfin, Zillow), county assessor records, and permit history for the facts that drive maintenance: year built, stories, square footage, construction, roof, foundation, heating and cooling type, water heater, public sewer vs. septic, fireplaces, garage, deck, trees, and climate risks. It collects property facts only, never owner or resident information.
2. **Report and confirm.** Summarizes what public records show, flags conflicts between sources, and asks the owner only the questions records can't answer, one at a time, starting with the highest-impact one (usually the number and type of HVAC systems). Every fact is tagged **[public]**, **[owner]**, or **[assumed]**.
3. **Draft the house files.** Writes a home profile, a month-by-month schedule chosen from a task library and shifted for the local climate zone, local cost research with sources, and a few house-specific "principles" that explain what really drives that home's upkeep.
4. **Assemble and deliver.** A Python script validates the schedule, builds the house skill, calendar, and reminders, packages the new skill, and calculates the annual cost totals, so the arithmetic is done by code rather than estimated by the AI.

## What's inside the skill

```
home-maintenance-builder/
├── SKILL.md                     Instructions Claude follows (the four steps above)
├── references/
│   ├── research-guide.md        Where to look, which fields matter, privacy rules, local cost searches
│   ├── task-library.md          Master list of maintenance tasks, when they apply, and national cost ranges
│   └── climate-regions.md       How to shift the schedule for cold, hot-humid, hot-dry, marine, and mixed climates
├── scripts/
│   └── new_house.py             Validates the schedule and assembles/packages the house-specific skill
└── assets/house-skill/          Template for the generated per-house skill
    ├── SKILL.md.tmpl            Instructions for the per-house skill (monthly to-dos, troubleshooting, costs)
    ├── references/home-profile.template.md
    ├── assets/calendar-template.html   The interactive calendar page
    └── scripts/
        ├── build_calendar.py    Builds the calendar and prints annual cost totals
        └── build_ics.py         Builds the monthly calendar-reminder file
```

This shows the assignment's main idea: a skill is more than one prompt. It combines **instructions** (`.md` files) with **scripts** (`.py` files) that collect, transform, and calculate data the same way every time.

## Data sources

All free and public:
- **Property facts:** Redfin, Zillow, and other listing sites; county assessor and property records; building permit history
- **Local prices:** web searches for local service costs (HVAC tune-ups, gutter cleaning, dryer vent cleaning, termite bonds, water heater flushes, etc.), with sources and the month checked recorded in the plan
- **Climate:** IECC climate zones and local average first and last frost dates

## How to use it

1. **Install:** in Claude, open the `.skill` file and click **Save skill**. To use it in Claude Code instead, unzip it into `~/.claude/skills/`.
2. **Run:** start a new chat and give it an address, for example *"Build a home maintenance plan for 123 Main St, Springfield, IL 62701."*
3. **Answer** a few quick questions about the house.
4. **Save** the generated house skill, open the calendar, and import the reminders file.

Each house gets its own skill, so a home, a rental, and a parent's house can all coexist.

## Notes

- Cost figures are **planning estimates, not quotes**, and cover routine maintenance only. Repairs and replacements are excluded.
- A skill can't send reminders on its own; the `.ics` calendar file (or a scheduled task) provides them.
