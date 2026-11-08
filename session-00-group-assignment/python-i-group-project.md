# Python I · Group Project
## Build a Tool People Would Actually Use

*A command-line application of your choice, built with everything from Sessions I–X, and pitched in 5 minutes with 3 slides.*

> ***"Find a real person with a real, everyday problem, build a Python tool that solves it, and convince us in five minutes."***

**Think of it as Sessions I–X in one project.** The Session X budget manager is your reference architecture: models, validators, storage, reports and a main program that only coordinates. This time, **you choose the problem.**

---

## 1. The Project at a Glance

One project, four ideas to keep in mind: who, what, how, and what you show at the end.

| 👥 Who | 🎯 What | 🛠️ How | 🎤 Output |
|---|---|---|---|
| You work **in groups**, as formed in class. Every member must be able to explain every part of the code. | A **tool of your choice** that solves a real problem for a real user, and keeps its data between runs. | **Python standard library only**: a `uv` project in Git, layered modules, classes with inheritance, JSON and CSV files, an `argparse` command-line interface. | A **5-minute pitch with exactly 3 slides** and a live demo, plus the code repository and a README. |

### The architecture you will build

Every project, whatever its topic, has the same layers under the hood. The user only talks to `main.py`; `main.py` only coordinates.

```
USER (terminal)
   │  plantpal add Aloe --kind succulent
   ▼
main.py ── argparse: parse the command, call the right functions, print the result
   │
   ├──► validators.py   clean and check every input (try/except, regex)
   ├──► models.py       your classes: a base class + subclasses (inheritance)
   ├──► storage.py      load / save JSON · export CSV · timestamped backups
   └──► reports.py      summaries, rankings, statistics
                │
                ▼
data/records.json  ·  exports/*.csv  ·  data/backups/*.json
```

### Technologies: required, optional and not accepted

| Area | Required | Optional, extra credit | Not accepted as a replacement |
|---|---|---|---|
| **Project** | `uv` project with `pyproject.toml`, Git repository on GitHub | tests with `unittest` or `pytest` | a loose `.py` file or a notebook |
| **Interface** | command-line interface with `argparse` subcommands | an interactive menu as a second mode, coloured output | a GUI or a website (that comes in Sessions XV–XVIII) |
| **Data** | JSON file as the store, CSV export or import | importing a messy CSV with a cleaning report | a database, or data that disappears when the program ends |
| **Libraries** | the **standard library** only (`json`, `csv`, `re`, `pathlib`, `datetime`, `argparse`, `statistics`…) | — | `pandas`, `requests`, `flask` or any third-party library in the core logic |

> ⚠ **Why only the standard library?** This project proves that you master the foundations. APIs, pandas, web apps and dashboards come in the second half of the course, and your pitch can say how you would use them next.

---

## 2. Choose Your Idea

You are **completely free to choose your topic**, as long as it passes the three tests below.

> ***"For [user], who struggles with [problem], our tool [does what] so that they can [benefit]."***

**Examples:**
- For **students sharing a flat**, who argue about who paid for what, our tool **records shared expenses and settles who owes whom** so that they can **close the month in one command**.
- For **people with many house plants**, who forget when each one needs water, our tool **tracks plants by type and watering frequency** so that they can **see today's to-do list in one command**.

### The three tests of a good idea

1. **A real user with a real problem.** You can name who would use it and what they do today without your tool (a spreadsheet, notes, memory).
2. **Data that lives over time.** The tool stores records that the user adds, lists, changes and deletes across many runs, and then asks questions about them.
3. **Room for the requirements.** Your records come in **at least two kinds** that behave differently (this is where inheritance goes), and there are at least **two useful reports** to compute.

⚠ **Avoid:** a calculator or a quiz with no stored data; a copy of the Session X budget manager (a money-related idea must add something substantially different, such as splitting between people or recurring subscriptions); ideas that need the internet or a real database to work.

### Ideas to get you started

| Idea | Two kinds of record | Reports that make it useful |
|---|---|---|
| 🏠 **Flat-share expenses** | `SharedExpense` / `PersonalExpense` | who owes whom, spending by category and person |
| 🌱 **Plant care** | `Succulent` / `Tropical` | what needs water today, most neglected plant |
| 🏋️ **Training log** | `CardioSession` / `StrengthSession` | weekly volume, personal records, longest streak |
| 📚 **Study planner** | `Exam` / `Assignment` | what is due this week, hours planned per course |
| 🥫 **Pantry and expiry dates** | `Fresh` / `Frozen` | what expires in the next 3 days, shopping list |
| 💼 **Job application tracker** | `Internship` / `FullTimeJob` | applications per stage, response rate, follow-ups due |
| 🎟️ **Student club events** | `Talk` / `Workshop` | attendance per event, most active members (with fake data) |
| 🔁 **Subscriptions manager** | `MonthlySub` / `YearlySub` | real monthly cost, renewals coming up |
| 🎸 **Practice log** (music, languages…) | `Lesson` / `SoloPractice` | minutes per week, streaks, goals reached |
| 📦 **Small shop inventory** | `Product` / `PerishableProduct` | low stock alerts, best sellers, stock value |

These are only examples: **your own idea is welcome and usually makes a better pitch.**

### Rules about data

- ⚠ **No real personal data.** Use invented names, emails and phone numbers in your sample data, never those of classmates or real people.
- Ship a **sample data file with at least 30 synthetic records** (`data/sample.json`, in the same format as `data/records.json`), and explain in the README how to start from it, for example `cp data/sample.json data/records.json`.
- Send your professor a **one-paragraph proposal** (user, problem, record types, reports) before you start coding.

---

## 3. Minimum Requirements: What Must Be Under the Hood

Your topic is free, but the engineering underneath is the same for every group.

Whatever your topic, your project **must** meet every point below. Each one comes from a session of the course, and your README must say **where** each requirement is met (file and function).

### Functional minimum

Your tool must let the user:

1. **Create** records, with every input validated.
2. **List and search** records, with at least one filter (for example by kind, category or date).
3. **Update** and **delete** a record.
4. Run **at least two reports** (summary, ranking, totals, upcoming items…).
5. **Export** data or a report to **CSV**.
6. Make a **timestamped backup** of the data file.

…and everything must **survive a restart**: close the program, run it again, and the data is still there.

### Technical minimum, session by session

| Session | Minimum requirement |
|---|---|
| **I** · Professional setup | `uv` project (`pyproject.toml`, `uv.lock`), `src/` layout, `.gitignore`, README. Git history with **commits from every member**. Functions with **type hints and docstrings**. |
| **II** · Control flow and errors | **The program never crashes on bad input**: wrong types, empty text, negative numbers, unknown IDs. Specific exceptions (`ValueError`, `KeyError`, `FileNotFoundError`…), never a bare `except:`. |
| **III** · Data structures | Choose the right structure for each job: at least one **dict** used as a lookup, one **set** (duplicates or unique values) and one **nested** structure (list of dicts, dict of lists). |
| **IV** · Pythonic idioms | At least **one list or dict comprehension**, one `sorted(..., key=lambda ...)`, and one use of `zip()` or `enumerate()`. No `for i in range(len(...))`. |
| **V** · Files | Data stored in **JSON**, loaded at start and saved after every change, handling a **missing or corrupted file**. **CSV** export (or import) with `csv.DictWriter` / `csv.DictReader`. |
| **VI** · Strings and regex | At least one **regular expression** that validates or cleans an input (a date, an email, a code, a phone number…), plus string normalisation (`strip`, `lower`, `title`). |
| **VII** · Command line | `argparse` with **at least 5 subcommands**, each with help text. `pathlib` for paths. A **backup** command that writes a timestamped copy. |
| **VIII** · OOP foundations | At least **two classes** with `__init__`, methods, `__str__` or `__repr__`, and `to_dict()` / `from_dict()` to save and load objects. |
| **IX** · Advanced OOP | An **inheritance hierarchy**: one base class and **at least two subclasses**, `super()`, a method **overridden** and called polymorphically, at least one `@classmethod` and one `@staticmethod`. |
| **X** · Architecture | Separate modules for **models, validators, storage, reports and main**. `main.py` coordinates and does not compute. |

> ✓ **A simple self-test:** a classmate clones your repository, runs `uv sync` and your README commands, and tries to break the tool with bad inputs. If nothing crashes and every command works, you have met the minimum.

### Going further (extra credit)

- **Tests** for validators and reports (`unittest` or `pytest`).
- **Import a messy CSV** and print a cleaning report (rows read, fixed, rejected, duplicates removed).
- **Statistics** with the `statistics` module (median, mean, standard deviation) in your reports.
- **ASCII charts** in the terminal (bars made of `█`) or coloured output.
- An **interactive menu mode** next to the subcommands, or **undo** of the last change.
- A **scheduled run** with cron or Task Scheduler (for example a daily reminder).
- A **config file** (JSON) for settings such as currency, default categories or file paths.

---

## 4. Project Structure and Code Skeleton

A common layout and a tested starting point, so that you spend your time on your idea.

### The folder layout

Create the project with `uv init --package your-tool` and organise it like this:

```text
your-tool/
├── pyproject.toml          # project metadata + the "your-tool" command
├── uv.lock
├── README.md               # team, problem, how to run, requirements map
├── .gitignore
├── data/
│   ├── records.json        # your data (created by the tool, not committed)
│   ├── sample.json         # ≥ 30 synthetic records, ready to try
│   └── backups/            # timestamped copies (not committed)
├── exports/                # CSV files produced by the tool
└── src/your_tool/
    ├── __init__.py
    ├── main.py             # argparse: parse → call → print
    ├── models.py           # base class + subclasses
    ├── validators.py       # input rules, regex, cleaning
    ├── storage.py          # JSON load/save, CSV export, backup
    └── reports.py          # summaries, rankings, statistics
```

In `pyproject.toml`, point the command to your `main()` so that `uv run your-tool ...` works from the project folder:

```toml
[project.scripts]
your-tool = "your_tool.main:main"
```

### Skeleton: the command-line interface

This is a minimal plant-care example. Adapt the idea, the commands and the names to your project. First, `main.py` imports the layers it coordinates:

```python
import argparse
from datetime import date

from plantpal.models import Plant, Succulent, Tropical
from plantpal import storage

KINDS = {"plant": Plant, "succulent": Succulent, "tropical": Tropical}
```

Then one function declares every subcommand, its arguments and its help text:

```python
def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(prog="plantpal", description="Never forget to water a plant again.")
    sub = parser.add_subparsers(dest="command", required=True)

    add = sub.add_parser("add", help="add a plant")
    add.add_argument("name")
    add.add_argument("--kind", choices=KINDS, default="plant")

    sub.add_parser("list", help="show every plant and its next watering")
    sub.add_parser("backup", help="copy the data file with a timestamp")
    return parser
```

And `main()` only coordinates: load, validate, call the right layer, print.

```python
def main() -> None:
    args = build_parser().parse_args()
    plants = storage.load_plants()                 # storage layer

    if args.command == "add":
        if not Plant.is_valid_name(args.name):     # validation before anything is saved
            print("Error: the name must have 1-40 characters.")
            return
        plants.append(KINDS[args.kind](args.name, date.today()))
        storage.save_plants(plants)
        print(f"Added {args.name}.")
    elif args.command == "list":
        for plant in sorted(plants, key=lambda p: p.next_watering()):
            print(plant.describe())                # polymorphism
    elif args.command == "backup":
        print(f"Backup saved to {storage.backup()}")
```

### Skeleton: a hierarchy you can save

The base class holds the data and the behaviour every plant shares:

```python
from datetime import date, timedelta


class Plant:
    kind = "plant"
    default_every = 7  # days between waterings

    def __init__(self, name: str, last_watered: date, every: int | None = None):
        self.name = name
        self.last_watered = last_watered
        self.every = every or self.default_every

    def next_watering(self) -> date:
        return self.last_watered + timedelta(days=self.every)

    def describe(self) -> str:
        return f"{self.name}: water every {self.every} days, next on {self.next_watering()}"
```

The trick for saving different subclasses in one JSON file is to store a `kind` field, and let a `@classmethod` rebuild the right subclass. A `@staticmethod` holds a rule that needs no object (one is enough here; most input rules belong in `validators.py`):

```python
    def to_dict(self) -> dict:
        return {"kind": self.kind, "name": self.name,
                "last_watered": self.last_watered.isoformat(), "every": self.every}

    @classmethod
    def from_dict(cls, data: dict) -> Plant:
        subclass = {c.kind: c for c in (Plant, Succulent, Tropical)}[data["kind"]]
        return subclass(data["name"], date.fromisoformat(data["last_watered"]), data["every"])

    @staticmethod
    def is_valid_name(name: str) -> bool:
        return bool(name.strip()) and len(name.strip()) <= 40
```

Subclasses change the defaults and override behaviour, so the same `describe()` call gives different results:

```python
class Succulent(Plant):
    kind = "succulent"
    default_every = 14

    def describe(self) -> str:  # override: same call, different behaviour
        return super().describe() + " (let the soil dry out)"


class Tropical(Plant):
    kind = "tropical"
    default_every = 3
```

### Skeleton: the storage layer

`storage.py` is the only module that touches files. A missing file means "no plants yet"; a corrupted one stops the program instead of overwriting the user's data:

```python
import json
import shutil
import sys
from datetime import datetime
from pathlib import Path

from plantpal.models import Plant

DATA_FILE = Path("data/records.json")
BACKUP_DIR = Path("data/backups")


def load_plants() -> list[Plant]:
    """Load every plant; a missing file just means no plants yet."""
    if not DATA_FILE.exists():
        return []
    try:
        records = json.loads(DATA_FILE.read_text(encoding="utf-8"))
    except json.JSONDecodeError:
        # stop, instead of overwriting the user's data with an empty list
        sys.exit(f"Error: {DATA_FILE} is corrupted. "
                 f"Restore a copy from {BACKUP_DIR}/.")
    return [Plant.from_dict(record) for record in records]
```

Saving rewrites the whole file after every change, and `backup()` copies it with a timestamp:

```python
def save_plants(plants: list[Plant]) -> None:
    """Write every plant to the JSON file."""
    DATA_FILE.parent.mkdir(parents=True, exist_ok=True)
    records = [plant.to_dict() for plant in plants]
    DATA_FILE.write_text(json.dumps(records, indent=2), encoding="utf-8")


def backup() -> Path:
    """Copy the data file to data/backups/ with a timestamp in its name."""
    if not DATA_FILE.exists():
        sys.exit("Nothing to back up yet: add a plant first.")
    BACKUP_DIR.mkdir(parents=True, exist_ok=True)
    stamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    target = BACKUP_DIR / f"records_{stamp}.json"
    shutil.copy2(DATA_FILE, target)
    return target
```

### Running the skeleton

```bash
$ uv run plantpal add Aloe --kind succulent
Added Aloe.
$ uv run plantpal add Monstera --kind tropical
Added Monstera.
$ uv run plantpal list
Monstera: water every 3 days, next on 2026-11-11
Aloe: water every 14 days, next on 2026-11-22 (let the soil dry out)
$ uv run plantpal add Cactus --kind cactus
usage: plantpal add [-h] [--kind {plant,succulent,tropical}] name
plantpal add: error: argument --kind: invalid choice: 'cactus' (choose from plant, succulent, tropical)
```

*The skeleton was tested with `uv` 0.12 and Python 3.14. It is deliberately incomplete: there is no update, delete, search, report, CSV export or regex yet. That is your job.*

---

## 5. The Pitch: 5 Minutes, 3 Slides

Five minutes to show a person with a problem, a tool that solves it, and proof that it works.

### The rules

- **5:00 is a hard stop.** Rehearse with a timer.
- **Exactly 3 slides.** The title of your project goes on slide 1; there is no separate cover, agenda or "thank you" slide.
- **A live demo is part of the pitch** (60–90 seconds, in the terminal). Prepare the commands in advance and keep a screenshot as a backup in case something fails.
- **Everyone speaks**, and after the pitch anyone may be asked about any part of the code.

### Slide 1 · The problem (≈ 1 minute)

- Your **one-sentence pitch**: "For [user], who struggles with [problem], our tool…".
- **Who** suffers the problem and **what they do today** (a messy spreadsheet, notes, nothing).
- **Why it matters:** time lost, money lost, things forgotten. One number or one story is enough.

### Slide 2 · How it works (≈ 2.5 minutes, demo included)

- The **architecture** of your tool: your layers and your class hierarchy, as one diagram.
- **One technical decision** you are proud of: a tricky regex, a clever report, how you handled bad input.
- The **live demo**: add a record, try a bad input, run a report, export to CSV.

### Slide 3 · Results and what's next (≈ 1.5 minutes)

- **What the tool does today:** your commands and reports, as a short checklist.
- **One thing that broke** and how you fixed it (a bug, a crash, a design you changed).
- **What's next:** how the second half of the course would make it better: an API, a Flask or FastAPI web version, a pandas or Streamlit dashboard.

> ✓ **A good pitch is a story:** a person with a problem (slide 1), a tool that solves it, seen working (slide 2), and proof that it is real and could grow (slide 3).

⚠ **Avoid:** reading your code line by line; slides full of text; a demo you have not rehearsed; going over time.

---

## 6. Deliverables and Evaluation

What you submit, how to submit it, and how it will be graded.

### What you deliver

| Deliverable | Format |
|---|---|
| 💻 **Code repository** | GitHub repository link, with `src/`, `pyproject.toml`, `uv.lock`, `data/sample.json`, and no personal data or secrets |
| 📄 **README** | team, the one-sentence pitch, how to install and run (`uv sync`, `uv run your-tool ...`), how to load the sample data, every command with an example, the folder structure, and the **requirements map** |
| 🎤 **The 3 slides** | PDF (plus the original file) |
| 🎬 **Demo backup** (optional) | screenshots or a short screen recording |

**The requirements map** is a table in your README that tells us where each minimum requirement lives:

| Requirement | Where |
|---|---|
| Inheritance + override | `models.py` → `Succulent.describe()` |
| Regex validation | `validators.py` → `validate_date()` |
| CSV export | `storage.py` → `export_csv()` |
| … | … |

- **Submit through Blackboard** before the due date your professor announces, as a single `.zip` named after your group, with the slides (PDF and original) and a text file with your repository link.
- ⚠ **Before you submit:** clone your own repository into a new folder, run `uv sync`, and try every command in the README. If it does not work for you, it will not work for us.

### Proposed rubric

| Criterion | Weight | Excellent looks like… |
|---|---|---|
| **Idea & problem** | 10% | A real user, a clear problem, records and reports that make sense for it |
| **Minimum requirements** | 35% | Every functional and technical point met, and easy to find thanks to the requirements map |
| **Code quality & architecture** | 20% | Clean layers, readable names, type hints, docstrings, no duplicated logic |
| **Robustness** | 10% | Nothing crashes: bad inputs, missing or corrupted files, empty data |
| **Git & README** | 10% | Commits from every member over time, a README that lets anyone run the tool in two minutes |
| **Pitch & demo** | 15% | A clear story in 5 minutes and 3 slides, a working live demo, every member can answer questions |
| **Extra credit** | + | Tests, messy CSV import, statistics, ASCII charts, scheduling, an original idea |

*Weights are a proposal; your professor confirms the final rubric.*

---

## 7. How to Work

A roadmap, roles and habits that keep a group project on track.

### A roadmap in five phases

| Phase | What you do | Done when… |
|---|---|---|
| **1 · Idea** | choose the user and problem, define the record types and reports | ✔ one-paragraph proposal approved |
| **2 · Skeleton** | `uv init --package`, Git repository, empty modules, `argparse` with every subcommand | ✔ `uv run your-tool --help` works for everyone |
| **3 · Core** | models with inheritance, JSON storage, create and list | ✔ data survives a restart |
| **4 · Features** | update, delete, search, validators with regex, reports, CSV export, backup | ✔ every minimum requirement met |
| **5 · Polish & pitch** | bad-input testing, README and requirements map, 3 slides, rehearsal | ✔ a 5-minute pitch with a working demo |

### Team roles

| Suggested role | Focus |
|---|---|
| **Product & pitch lead** | the user and problem, the README, the 3 slides, timing the rehearsal |
| **Models lead** | the class hierarchy, `to_dict` / `from_dict`, polymorphism |
| **Data & CLI lead** | `storage.py`, `main.py`, `argparse`, CSV export, backups |
| **Quality lead** | `validators.py`, regex, bad-input testing, `reports.py` |

Everyone still writes code and commits it, and everyone must understand the whole project.

### Practical tips

- **Agree on the record format first** (the dictionary each object becomes in JSON). Everything else depends on it.
- **Commit small and often**, with clear messages. Pull before you push.
- **Never commit** `data/records.json`, `data/backups/` or real personal data: add them to `.gitignore`.
- **Write the sample data early**, so that reports have something to compute from day one.
- **Try to break your own tool** before anyone else does: empty strings, letters where numbers go, an ID that does not exist, a deleted data file.

### Checklist before you submit

- ✅ **It runs from a fresh clone:** `uv sync` and every README command works.
- ✅ **Every functional minimum works:** create, list/search, update, delete, two reports, CSV export, backup.
- ✅ **Data survives a restart**, and a missing or corrupted JSON file does not crash the program.
- ✅ **Bad inputs never crash it:** each one gets a clear message.
- ✅ **The hierarchy is real:** a base class, two subclasses, an override used polymorphically, a `@classmethod` and a `@staticmethod`.
- ✅ **At least one regex** validates or cleans an input.
- ✅ **Only the standard library** in the core logic.
- ✅ **Commits from every member**, and a README with the requirements map.
- ✅ **No personal data** of real people, only synthetic samples.
- ✅ **The pitch fits in 5:00** with exactly 3 slides, and the demo has been rehearsed.

### Frequently asked questions

| Question | Answer |
|---|---|
| **Can two groups have the same idea?** | Yes, but the pitches will be compared, so make yours different: a specific user, original reports. |
| **Can we use pandas, requests or another library?** | Not in the core logic: the project is about the standard library. Mention them in your "what's next". |
| **Can we build a menu instead of `argparse`?** | `argparse` subcommands are required. An interactive menu can be an extra mode. |
| **Can we build a GUI or a website?** | Not for this project. Web apps and dashboards come later in the course. |
| **How much code is enough?** | Enough to meet every minimum requirement cleanly. Quality beats quantity. |
| **Can we use AI assistants?** | Yes: critical use of GenAI is encouraged. Disclose every tool you used in the README, follow IE's academic integrity rules, and be ready to explain every line. |
| **Does the cover count as a slide?** | No separate cover: your project's name goes on slide 1, and the deck has exactly 3 slides. |
| **What if the demo fails live?** | Show your backup screenshots, and keep going. Rehearsing the demo makes this unlikely. |

---

## Quick Reference: The Project at a Glance

```
┌──────────────────────────────────────────────────────────────────┐
│  IDEA        a real user · a real problem · data over time       │
│  RECORDS     1 base class + ≥ 2 subclasses (inheritance)         │
│  FEATURES    create · list/search · update · delete              │
│              ≥ 2 reports · CSV export · timestamped backup       │
│  LAYERS      main · models · validators · storage · reports      │
│  DATA        JSON store · ≥ 30 synthetic records · no real PII   │
│  ROBUST      bad input never crashes · missing file is fine      │
│  PYTHONIC    comprehension · sorted(key=lambda) · zip/enumerate  │
│  REGEX       ≥ 1 validation or cleaning pattern                  │
│  CLI         argparse · ≥ 5 subcommands · pathlib                │
│  PROJECT     uv · src/ layout · Git with every member · README   │
│  LIBRARIES   standard library only                               │
│  PITCH       5:00 · exactly 3 slides · live demo                 │
│  DELIVER     repo + README (requirements map) + slides (PDF)     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Questions? Good luck!

Find a real problem, build a tool that solves it, and show it working in five minutes.

