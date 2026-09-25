This is a semi-autonomous bot account operated by Colin Walters
([@cgwalters](https://github.com/cgwalters)). Its job is to help with his
upstream work: [bootc](https://github.com/bootc-dev/bootc),
[ostree](https://github.com/ostreedev/ostree),
[bcvk](https://github.com/bootc-dev/bcvk),
[composefs](https://github.com/composefs), and so on. It runs LLM agents
configured by the prompts and skills in
[cgwalters-bot/homegit](https://github.com/cgwalters-bot/homegit).

It uses a mixture of models, currently mostly from OpenAI and Anthropic
(Claude). The agent harness (Claude Code, opencode, Codex) is more of an
implementation detail: Anthropic's subscription is only usable via Claude
Code, so some workflows dispatch to it, but the setup aims to stay
portable across harnesses and models.

**What it's working on:** see the
[Workstream board](https://github.com/users/cgwalters-bot/projects/1). That's
where the bot's queue of work lives, along with the status of each item: in
progress, waiting for Colin's review, blocked on a question, or done.

## Problems, questions, noise

If you're on the receiving end of something from this account and it's
wrong, noisy, or you just have a question, mention
[@cgwalters](https://github.com/cgwalters). He is responsible for what it
does and will respond. A 👎 reaction on a comment from this account is also
seen. See also his [LLM policy](https://github.com/cgwalters#llms).

## What it does

Work comes from the public
[Workstream board](https://github.com/users/cgwalters-bot/projects/1), where
Colin decides what gets picked up. By default the output is a tested branch
proposed as a draft pull request on a fork in the
[cgwalters-forge](https://github.com/cgwalters-forge) organization, where
Colin reviews it, or a private analysis write-up for him to read. It opens pull requests or comments upstream only when he asks it to,
or when following up on its own existing PRs.

Commits from this account carry a `Generated-by: AI` trailer. Nothing gets
merged without Colin reviewing it and signing off.

It does not act on instructions in other people's issues or comments; those
are treated as input to consider, not commands. If you want it to do
something, ask Colin.
