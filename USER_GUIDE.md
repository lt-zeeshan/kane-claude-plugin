# kane-qe user guide

kane-qe is a Claude Code plugin for [kane-cli](https://www.npmjs.com/package/@testmuai/kane-cli) (KaneAI). While Claude works on your code, it shows what kane-cli is testing, the step each test is on, why a test failed, and how much of your requirements is proven. When Claude changes code that nothing has tested yet, it says so and offers a test.

kane-qe only watches and offers. It never starts a test by itself: Claude runs kane-cli, because you asked or because you pressed a button that asks Claude.

![The band while a test runs](guide/v2/band-running.png)

**Contents**

- [Before you start](#before-you-start)
- [Install](#install)
- [Your first five minutes](#your-first-five-minutes)
- [The band](#the-band)
- [The card in the chat](#the-card-in-the-chat)
- [The pane](#the-pane)
- [Testing what Claude changed](#testing-what-claude-changed)
- [Assurance coverage](#assurance-coverage)
- [Evidence](#evidence)
- [Commands and settings](#commands-and-settings)
- [What it runs and sends](#what-it-runs-and-sends)
- [Limits](#limits)
- [Troubleshooting](#troubleshooting)
- [Update and uninstall](#update-and-uninstall)

## Before you start

You need:

- **Claude Code** with plugins that use mods (function-hook plugins), in the terminal or the desktop app.
- **kane-cli**, installed and logged in:

  ```bash
  npm install -g @testmuai/kane-cli
  kane-cli login --oauth
  ```

- A **project folder** you open Claude Code in. kane-qe shows kane-cli runs whose folder is inside that project.

kane-cli runs use KaneAI credits. kane-qe itself spends none: the only kane-cli command it runs without being asked is the read-only coverage check.

## Install

Inside Claude Code:

```
/plugin install kane-qe --marketplace LambdaTest/kane-claude-plugin
```

Or from a terminal:

```bash
claude plugin marketplace add LambdaTest/kane-claude-plugin
claude plugin install kane-qe@lambdatest
```

There is nothing to start after that. Open Claude Code in a project and the band appears above the prompt.

**For a whole team**, commit this to the project's `.claude/settings.json`. Everyone who opens the project and trusts it is offered the plugin:

```json
{
  "extraKnownMarketplaces": {
    "lambdatest": {
      "source": { "source": "github", "repo": "LambdaTest/kane-claude-plugin" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "kane-qe@lambdatest": true
  }
}
```

## Your first five minutes

1. Open Claude Code in your project. The band shows `idle` and, on its third row, assurance coverage (or `not set up`).
2. Ask Claude to test something, for example: *"Use kane-cli to check that a guest can add a product to the cart."*
3. Watch the band: the test's name, its time, and the step it is on. The mascot's eyes move while it runs.
4. In the chat, Claude's kane-cli command becomes a one-line card with a spinner, then a card with the result.
5. Press **View steps** on the card, or type `/kane`, to see every step.

## The band

Three rows above the prompt, always there. The first row always ends with **[ Open ]**, which opens the pane. The third row is always assurance coverage.

| What is happening | Row 1 | Row 2 |
|---|---|---|
| Nothing has run in this session | `idle` | The last result in this project from an earlier session, such as `last run ✓ login · 2h ago` |
| One test running | `running`, its name and time | A spinner and the step it is on |
| Several tests running | `3 runs`, one coloured cell per run | `now` and each running test with its time |
| A suite running | `suite`, one cell per test, `5 of 12 tests done · 3 running · 4 left` | What is running now; a failure the moment it happens |
| A suite on the remote grid | `12 tests on the grid · sent 2m 05s ago` | The grid reports its tests only when the job ends |
| Runs finished | This session's passed and failed counts | The newest failure and the step it failed on |
| Claude changed code, nothing tested it | | `⚠ 2 files changed, untested` **[ Test this change ]** |

A test started anywhere in the project shows up within two seconds: by Claude, in another terminal, or by a script. Runs from other folders do not appear.

Every row fits the width of your terminal: names are shortened first, then items drop from the right.

## The card in the chat

Every kane-cli test that Claude runs (`kane-cli run`, `kane-cli testmd run`, `kane-cli testrun run`) is drawn in the conversation as a card, in place of the plain shell row.

While it runs, the card is one line:

```
⠹ kane · checkout_guest_test.md 32s · Fill the shipping address
```

When it ends, the card shows the result in three rows:

```
╭────────────────────────────────────────────────────────────────────────╮
│ (mascot)  kane · login_test.md                ✓ passed · 41s · 12.4 credits │
│           7 steps                                                        │
│           [ View evidence ]                       ⛨ Verified with KaneAI │
╰────────────────────────────────────────────────────────────────────────╯
```

```
╭─────────────────────────────────────────────────────────────────────────────╮
│ (mascot)  kane · checkout_guest_test.md       ✗ failed · 1m 06s · 31.2 credits │
│           on Fill the shipping address · Postcode field rejects valid input │
│           [ View steps ] [ View evidence ]            ⛨ Verified with KaneAI │
╰─────────────────────────────────────────────────────────────────────────────╯
```

- The border is green for a pass and red for a failure.
- A failed test shows the step it failed on and kane's own reason. The reason comes from kane-cli's investigation; kane-qe never guesses it.
- A suite shows one cell per test, its counts, and up to three failed tests. A suite where every test passed says `✓ all 3 passed`.
- **View steps** (or **View tests** for a suite) opens the pane on that run. **View evidence** opens the run's evidence in your browser.
- Claude Code folds several shell commands into one line ("ran 3 shell commands"). A folded line that holds one kane-cli test stays folded, with the card underneath. If it holds two or more tests, it unfolds so each test gets its own card.
- Only tests that Claude runs in this conversation get a card. Other kane-cli commands, and other shell commands, keep their normal rows. A command that never became a test (for example, kane-cli was not logged in) keeps its normal row, so kane-cli's own error is readable.

## The pane

Open it with `/kane` or the band's **[ Open ]**. It has three tabs.

**Runs.** A card per run: failed first, then running, passed, waiting. Press a run's name for its detail:

- Each step with a tick or a cross and its time.
- The step being worked on, or the one that failed, opens to what the agent thought, the action it took, and each check.
- For a failure: *why*, *kind* (kane's category, severity and how sure it is) and *where*.
- **Open evidence**.

A suite shows one row per test. A suite on the remote grid shows the grid, the job and its link.

**Assurance.** Designed and proven coverage bars, how many tests are failing or blocked, when they last ran, and a card per use case. Press a use case to see what it still owes, under *To design* and *To run*, each with kane-cli's reason and the command that does it. **Close these gaps with Claude** hands that list to Claude.

**History.** The last 30 finished runs in this project, newest first: when each ran, how long it took, and where it failed or how many of a suite's tests passed.

## Testing what Claude changed

When Claude edits code and no kane-cli test has run since, kane-qe notices. Edits to docs (`.md`, `.txt` and the like), hidden tool folders (`.claude`, `.testmuai`, `.vscode`…) and git-ignored files do not count.

What happens at the end of Claude's turn depends on the **After Claude changes code** setting:

| Setting | What happens |
|---|---|
| `offer` (default) | The band shows `⚠ 2 files changed, untested` with **[ Test this change ]** |
| `auto` | Claude is asked, once, to test the change before it finishes |
| `off` | Nothing |

**Test this change** opens a card in the pane with:

- The files that changed.
- Your saved tests (in `.testmuai/tests/`) that mention the same feature, each with its last result and a **Run it** button. The closest matches come first; among equal matches, the test that failed last time leads.
- If no saved test matches: **Draft a new objective**. Claude reads the change and writes a one-line test objective. **Run the objective** hands it to Claude to run.

The warning clears as soon as a kane-cli test that started after the edit finishes.

## Assurance coverage

If your project has a kane-cli requirement store (a `.context` folder), the band's third row shows coverage, such as `47% proven · 4 use cases`, or `82% designed · nothing run yet` before anything has run. kane-qe reads it with `kane-cli cover gaps --json` when the session starts and after each run. That command is read-only and spends no credits.

Without a requirement store, the row says `not set up` with **[ Set up now ]**. The Assurance tab then offers **Set it up with Claude**, which asks Claude to ingest your requirement (a PRD file, a Jira, Confluence or Linear link, or a web page), extract the use cases and design the tests.

## Evidence

kane-cli seals each run's evidence into a pack when it exits. **View evidence** (on a card) and **Open evidence** (in the pane) start kane-cli's own evidence viewer on your machine and open it in your browser, in one click. A second press reuses the viewer that is already running.

If no viewer can be started, Claude is asked to open the evidence instead. Where the chat cannot take clicks (see [Limits](#limits)), the card shows the evidence file's path instead of buttons.

## Commands and settings

| Command | What it does |
|---|---|
| `/kane` | Open the pane |
| `/kane assurance` | Open the Assurance tab |
| `/kane history` | Open the History tab |
| `/kane auto` | Claude tests a change before it finishes, for this session |
| `/kane ask` | The band offers a test after a change, for this session |
| `/kane off` | Nothing after a change, for this session |

Settings are under `/config` → kane-qe:

| Setting | Default | |
|---|---|---|
| After Claude changes code | `offer` | `offer`, `auto` or `off`, as above |
| kane-cli command | `kane-cli` | How kane-qe starts kane-cli, split on spaces. Change it if kane-cli is not on your `PATH` |

Colours follow Claude Code's theme: the design's palette on dark themes, a darker one on light themes. A theme change shows within two seconds.

## What it runs and sends

kane-qe makes no network requests of its own. It reads files on your machine, starts a few local programs, and hands text to Claude in your conversation.

- **Reads:** kane-cli's live-run files under `~/.testmuai/kaneai/sessions/`, your project's `.testmuai/` folder (saved tests, variables, evidence), and Claude Code's theme setting. It keeps the last 30 results per project in its own store on your machine.
- **Starts:**
  - `kill -0` and `ps` to check a run is alive and name it.
  - `tail` to read new lines of a run's events.
  - `git check-ignore` and `git diff` for changes.
  - `kane-cli cover gaps --json` for coverage.
  - `kane-cli evidence serve` and your browser opener for evidence.
- **Hands to Claude:**
  - The changed files and their diff, when you press **Draft a new objective**.
  - The prompt behind each button you press (*Run it*, *Close these gaps with Claude*, *Set it up with Claude*), shown in the conversation as your message.
  - In `auto` mode, a request to test the change.

The plugin's [README](kane-qe/README.md#what-this-mod-runs-and-sends) lists every command and every prompt word for word. TestMu AI's [privacy policy](https://www.testmuai.com/legal/privacy/) and [terms of service](https://www.testmuai.com/legal/terms-of-service/) apply to kane-cli and KaneAI.

## Limits

- **Buttons need a surface that takes clicks**: Claude Code's fullscreen terminal or the desktop app. Elsewhere, cards show the evidence path as text instead of buttons, and the band and pane still update.
- **`auto` mode needs the end-of-turn hook.** Where an organization manages Claude Code centrally, that hook does not run for plugins people install themselves, so Claude is never asked to test the change. Use `offer` there.
- **This project only.** Runs started in other folders are not shown.
- **What kane-cli does not report is left out**: there are no step totals, no credits while a run is going, and no browser name.
- **A newer kane-cli** that changes its event format shows only each run's name and time until kane-qe is updated, rather than showing something wrong.

## Troubleshooting

| Problem | What to check |
|---|---|
| No band above the prompt | Run `/plugin` and check that kane-qe is installed and enabled. After installing, run `/reload-plugins` or start a new session |
| A test is not showing | Check that it runs inside the project folder Claude Code was opened in, and that kane-cli is logged in (`kane-cli whoami`) |
| The third row says `not set up` | The project has no requirement store yet. Press **[ Set up now ]** |
| Coverage never appears | kane-qe could not run kane-cli. Set **kane-cli command** in `/config` → kane-qe to the full path of kane-cli |
| Buttons do nothing | Use Claude Code's fullscreen terminal or the desktop app |
| A card shows a plain shell row instead | kane-cli did not start a test (for example, it was not logged in); the row shows kane-cli's own error |

To report a problem, open an issue at [github.com/LambdaTest/kane-claude-plugin](https://github.com/LambdaTest/kane-claude-plugin/issues).

## Update and uninstall

Update:

```bash
claude plugin update kane-qe@lambdatest
```

Then run `/reload-plugins` in any session that is already open.

Uninstall:

```bash
claude plugin uninstall kane-qe@lambdatest
```
