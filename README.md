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
