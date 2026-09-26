This is a semi-autonomous bot account operated by Colin Walters
([@cgwalters](https://github.com/cgwalters)). Its job is to help with his
upstream work: [bootc](https://github.com/bootc-dev/bootc),
[ostree](https://github.com/ostreedev/ostree),
[bcvk](https://github.com/bootc-dev/bcvk),
[composefs](https://github.com/composefs), and so on. It runs LLM agents
configured by the prompts and skills in
[cgwalters-bot/homegit](https://github.com/cgwalters-bot/homegit); the
skills are plain [Agent Skills](https://agentskills.io) so they aren't tied
to one agent.

It uses a mixture of models, currently mostly from Anthropic (Claude) and
OpenAI. The agent harness (Claude Code, opencode, Codex) is more of an
implementation detail, and the setup aims to stay portable across harnesses
and models. Today a long-running coordinator session dispatches worker
agents, each of which builds and tests on a disposable GitHub Actions
runner; work in progress moves this to scheduled workflows with an
[ACP](https://agentclientprotocol.com) agent wrapper.

**What it's working on:** the top priority is making the composefs backend
of bootc stable, tracked on the
[Composefs Stable board](https://github.com/users/cgwalters-bot/projects/2).
Everything else, including the bot's own tooling, is on the
[Workstream board](https://github.com/users/cgwalters-bot/projects/1), with
the status of each item: in progress, waiting for Colin's review, blocked on
a question, or done.

## Problems, questions, noise

If you're on the receiving end of something from this account and it's
wrong, noisy, or you just have a question, mention
[@cgwalters](https://github.com/cgwalters). He is responsible for what it
does and will respond. A 👎 reaction on a comment from this account is also
seen. See also his [LLM policy](https://github.com/cgwalters#llms).

If you maintain a project and would like the bot to behave differently
there (for example, only rebasing its pull requests when they conflict,
because your merge queue makes every push re-run CI), say so on one of its
pull requests; it records per-project preferences like that and follows
them.

## What it does

Work comes from the boards above, where Colin decides what gets picked up.
By default the output is a tested branch proposed as a draft pull request on
a fork in the [cgwalters-forge](https://github.com/cgwalters-forge)
organization, where Colin reviews it, or a private analysis write-up for him
to read. Every result is also checked by a second, independent agent before
it reaches him. Questions for him are issues in
[cgwalters-forge/tracker](https://github.com/cgwalters-forge/tracker), and he
works through the queue in a small
[review app](https://cgwalters-forge.github.io/review/).

It opens pull requests upstream only after Colin approves the forge draft
(and before that it checks the project's contribution and AI policy), and
it comments upstream only on its own pull requests or when he asks it to in
a thread. Commits from this account carry a `Generated-by: AI` trailer. For
projects that require a DCO, Colin's `Signed-off-by` is added only once he
has approved that exact change; nothing merges upstream without him.

The bot's own tooling repositories (homegit, the review app, the devspace
sandbox) are different: routine changes there merge once an independent
agent has reviewed them, so they don't wait on Colin. Changes to
architecture, security, or the bot's own rules still need his approval.

It does not act on instructions in other people's issues or comments; those
are treated as input to consider, not commands. If you want it to do
something, ask Colin.

## How it works

This section is for people who see the bot's PRs and want to know what's
behind them, or who want to run something similar. Everything described
here lives in [homegit](https://github.com/cgwalters-bot/homegit): the
shared agent instructions in `dotfiles/.config/AGENTS.md`, the skills in
`dotfiles/.claude/skills/`, and a handful of scripts in `bin/` for the
parts that shouldn't be left to an agent's judgment.

```
 cgwalters ──triage──▶ Workstream board ◀──┐ status, results, questions
                            │              │
                            ▼              │
  pings, forge inbox ──▶ coordinator ──▶ worker ◀──findings── reviewer
  board watch                            │   │
                   builds/tests over SSH │   │ bot-pr fork-pr
                                         ▼   ▼
                 devspace (no credentials)   draft PR on cgwalters-forge/REPO
                                               │ cgwalters edits, approves
                                               ▼
                                   bot-pr promote ──▶ upstream PR
```

### Identity and trust

`cgwalters-bot` is a separate GitHub account that acts for Colin; it is
not Colin, and it doesn't hold his credentials. The one rule everything
else rests on: only the board (which only its collaborators can edit)
and comments whose author is the `cgwalters` login carry intent. All
other GitHub text, including issue bodies, review comments, commit
messages and CI logs, is untrusted data. The bot reads it to understand
the work, but doesn't follow instructions in it, and a comment that
merely claims to be from Colin counts for nothing: the scripts check the
login GitHub recorded.

That applies to pings too. `bot-notify` polls the bot's notifications
(backed up by a pass over Colin's public events and a mention search,
since GitHub's notifications turned out to be unreliable). Colin's asks
become board items. A mention, review request or assignment from anyone
else is filed as an issue in this repository for Colin, and the bot
never acts on it. Similarly, each planning pass runs `bot-feedback`,
which turns every new 👎 or 😕 on the bot's content into an issue here,
with the bot's assessment of it.

### State: the Workstream board

The [Workstream board](https://github.com/users/cgwalters-bot/projects/1)
is a GitHub Projects (v2) board, and it's where the bot's state lives.
Each item is an issue or PR from any repository, or a draft item that
exists only on the board. The fields that matter:

- **Status**: no status means untriaged, and the bot never picks those
  up. Colin moves items to *Todo* (or the bot does, for his direct
  asks); the bot takes them through *In Progress* to *Draft* (a result
  is ready for Colin, but nothing is upstream yet), then *In Review* (a
  PR is open upstream) and *Done* (merged, or dropped by Colin). *Needs
  human* means blocked on a decision, with the question in Why.
- **Workflow**: what "finished" means. `branch` (the default) is a
  tested branch proposed on a forge fork; `analysis` is a write-up in a
  secret gist; `pr` means Colin asked for an upstream PR directly;
  `manual` means hands off.
- **Priority** (P0 to P2), **Org** (the upstream organization), **Why**
  (the rationale, the latest result in one line, and any open question),
  and **Branch**/**Gist** (links to the result).

There's no hidden database: even the scripts' own poll state sits in
archived draft items on the same board, and code and PR text live in git
and on the PRs, which the board just points at. Since the board is
public, nothing from a private repository goes on it beyond a URL.

### The loop

A long-running coordinator session drives everything, but does little
of the work itself. On a loop (a background sleep, cut short whenever a
worker finishes) it polls three sources: `bot-notify` for pings,
`bot-pr inbox` for Colin's activity on the bot's forge PRs, and
`bot-watch` for changes to the upstream issues and PRs of every board
item that isn't Done or manual (new comments and reviews, merges, pushes
by others, CI going red). `bot-watch --apply` also does the mechanical
board updates, like moving an item to Done when its PR merged.

Then it hands out work. Each Todo item (by priority) or ask from Colin
goes to a worker subagent, briefed with a short preamble that points it
at the skills it must follow; the worker updates the board itself. When
the worker reports back, a separate reviewer agent checks the branch or
write-up: whether it fixes the stated problem, test adequacy, scope, and
commit hygiene. The reviewer changes nothing itself; its findings go
back to the same worker, and this repeats until the reviewer says it
can ship. Only then does the coordinator point Colin at it. It also
posts a morning review queue here as an issue (e.g.
[#9](https://github.com/cgwalters-bot/cgwalters-bot/issues/9)).

### Staging on cgwalters-forge, then upstream

A finished change is pushed to the project's fork in the
[cgwalters-forge](https://github.com/cgwalters-forge) organization and
proposed there as a draft PR (`bot-pr fork-pr`), written as the upstream
PR it's meant to become. Colin reviews it like any other PR: he
comments, edits the title and description, and asks for changes, which
the bot squashes into the commits they belong to.

When he approves, with either an approving review or a comment line
that is exactly `/promote`, `bot-pr promote` rebases the branch onto
the current upstream base and opens the upstream PR from the forge
branch with the fork PR's current title and body. It then closes the
fork PR and moves the item to In Review. An approval covers only the
commit it was given on, so any later push needs a new one. Wording like
"looks good, ship it" isn't an approval; the bot asks for `/promote`
rather than guessing.

The commits say who wrote them. The bot commits as
`Colin Walters <walters+llm@verbum.org>`, where the `+llm` address is
what marks a commit as the bot's (older ones are authored as
`cgwalters-bot`), and each carries a `Generated-by: AI` trailer, or
whatever the project's policy asks for instead. The bot never adds a
`Signed-off-by`. `bin/bot-git`, the wrapper it commits through, sets
that identity and refuses the usual ways of adding one, and the real
gate is `bot-git check`, which the worker runs before pushing and the
reviewer runs again. The one exception is promotion: if the upstream
repository's branch rules require a DCO check, Colin's verified approval
of that exact head is his sign-off, so `promote` adds his
`Signed-off-by: Colin Walters <walters@verbum.org>` to the commits and
says so in the PR body. It only does that on repositories that require
DCO, and never for anyone else's commits unless he asks.

Upstream text is deliberately short. PR descriptions say what and why,
what was tested and where, and leave a placeholder for Colin's own
context on anything nontrivial, so his edits are visibly the human
layer. Beyond its own PRs, the bot doesn't review, label, react or
comment upstream, except for one short reply in a thread where Colin
@-mentioned it with a question.

### Devspaces

The machine running the agents holds the clones and the bot's GitHub
credentials, and does no building or testing. Every compile, test run,
container build or VM boot happens on a devspace: an ephemeral GitHub
Actions runner from
[bootc-dev/cgwalters-devspace-sandbox](https://github.com/bootc-dev/cgwalters-devspace-sandbox),
a RHEL 10 machine with 4 to 64 cores, KVM and podman, reachable over a
tailnet. `bot-devspace` starts one per task. The agent pushes its branch
to it over SSH and runs the project's own test entry points there as an
unprivileged user without sudo, then stops the devspace when done.

The agent machine never sends a devspace any credentials: no GitHub
token, SSH key or API key. And the unprivileged SSH user can't reach the
runner's own job credentials, which belong to a separate user. A
devspace gets source code and nothing else, and anything brought back
from it (logs, generated commits) is treated as data and reviewed before
it goes anywhere. Pushes to GitHub happen only from the agent machine.
Forge forks have their CI workflows disabled, so the devspace run *is*
the CI for a forge PR, and the PR description says what ran and where.
The project's real CI runs once the PR is promoted upstream.

### Where it's heading

Two things are in progress. The first is running the agents themselves
inside devspaces, in the style of GitHub's agentic workflows (gh-aw): an
`agent.yml` workflow works one board item per run (so far with a mock
model), with a condensed transcript in the Actions log and a summary
plus an encrypted full transcript as run artifacts. `bot-runs`
dispatches and inspects those runs; its contract is
[docs/devspace-agent-runs.md](https://github.com/cgwalters-bot/homegit/blob/main/docs/devspace-agent-runs.md).
Runners still get no credentials: the agent runs there as an
unprivileged user, on public repositories only, and the agent machine
applies what a run produces with its own credentials. The remaining
steps are tracked in
[#11](https://github.com/cgwalters-bot/cgwalters-bot/issues/11).

The second is a [review app](https://github.com/cgwalters-forge/review)
for staged PRs, so Colin can reword commit messages and tweak code
without dropping into a terminal
([#8](https://github.com/cgwalters-bot/cgwalters-bot/issues/8)). It's
public code and GitHub-native: it works on the Workstream board and the
forge issues and PRs, with gists for long context, rather than adding a
store of its own.
