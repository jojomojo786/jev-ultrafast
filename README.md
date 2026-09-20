# Jev Ultrafast — Usage Guide

My local copy of [browser-use/jev-ultrafast](https://github.com/browser-use/jev-ultrafast) (MIT license).
An ultrafast browser agent: give it one natural-language goal and it drives your real Chrome.
Original project documentation: [docs/ORIGINAL_README.md](docs/ORIGINAL_README.md).

## How it works (short version)

1. The page is read as an indexed table of visible elements (`[1] button ...`, `[2] combobox ...`).
2. TypeSafe's Jev model picks an **operation** (`CLICK`, `TYPE_TEXT`, `SELECT`, `SCROLL`, `WAIT`, `DONE`) and a **target element** in one request.
3. A small text LLM is only called when text needs to be typed.
4. Repeat until `DONE`.

No screenshots in the loop, one request per step — that's why it's fast and cheap (~$0.004 for a flight search).

## Requirements

- macOS with **Google Chrome** running
- [uv](https://docs.astral.sh/uv/)
- Two API keys (see below)

## Setup

```bash
git clone <this-repo>
cd jev-ultrafast
uv sync
cp .env.example .env
```

Edit `.env`:

```bash
TYPESAFE_API_KEY=...        # TypeSafe key — drives operation/target decisions
TYPESAFE_MODEL=jev-latest
TEXT_MODEL_API_KEY=...      # OpenRouter key — writes text when TYPE_TEXT is chosen
TEXT_MODEL_BASE_URL=https://openrouter.ai/api/v1
TEXT_MODEL=inception/mercury-2.5
TEXT_MODEL_REASONING=none
```

Then connect Chrome:

1. Open `chrome://inspect/#remote-debugging` in Chrome
2. Enable **"Allow remote debugging for this browser instance"**
3. Run `uv run browser-harness mac-approve` when the approval prompt appears
4. Verify with `uv run browser-harness --doctor` (daemon alive + active connection = OK)

## Usage

### 1. Interactive demo (recommended first run)

```bash
uv run jev
```

Open http://127.0.0.1:8766 → **Start demo → Run automatically**.
The inspector shows numbered elements, operation probabilities, and each executed action.
**Choose next** pauses before each step so you can watch it decide.

### 2. Run any task from the CLI

```bash
uv run --env-file .env python examples/run.py \
  --url https://en.wikipedia.org/wiki/Main_Page \
  --goal 'Find and open the Wikipedia article about Gödel’s incompleteness theorems.'
```

Prints elapsed time per step and the final URL. (Verified working: ~5 s, 3 actions.)

### 3. The Google Flights demo

```bash
uv run --env-file .env python examples/flights.py --keep-open
```

Searches Zürich → London flights (~7 s), independently verifies the result, and saves a trace.

### 4. Use it as a library

```python
from jev_ultrafast import Agent

with Agent(
    "https://www.google.com/travel/flights?hl=en",
    "Find one-way flights from Zurich to London on September 20, 2026, "
    "for one adult in economy. Stop when matching flight options are visible.",
) as agent:
    for state in agent.run():
        print(state["elapsed_ms"], state["status"])
```

Run your script with:

```bash
uv run --env-file .env python your_script.py
```

## Tips & limits

- **It uses your real Chrome profile** — you're already logged into your accounts. Powerful, but be careful with goals.
- **Write good goals:** one clear objective + a stop condition ("Stop when X is visible").
- **Verification:** a `DONE` decision isn't proof — check the final page yourself for important tasks.
- **Known limits (MVP):** shadow DOM, iframes, canvas, file uploads, pop-up tabs, nested scrolling, and complex keyboard widgets are not supported.
- If the browser disconnects: `uv run browser-harness --doctor` to diagnose, `uv run browser-harness --reload` to restart the daemon.

## Development

```bash
uv run ruff check .
uv run pytest        # offline tests
uv build
```

## Credits

Built by [Browser Use](https://github.com/browser-use) × [TypeSafe](https://docs.typesafe.ai/introduction).
This repo is a personal copy for local use — all credit to the original authors. MIT License.
