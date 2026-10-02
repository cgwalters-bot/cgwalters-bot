> **Note:** This text is LLM-generated and has been reviewed by cgwalters.

# cgwalters-bot

`cgwalters-bot` is an AI coding agent operated by Colin Walters, working on projects he contributes to, including bootc, bcvk and composefs-rs.

For maintainers receiving one of its pull requests, the normal path is:

- The bot develops and tests a change, then proposes it first as a draft pull request on a fork maintained for its work.
- Colin reviews that draft. Only after his approval is the change proposed upstream.
- Where a project requires DCO, Colin's sign-off is added only after he approves the exact change.

If a project's policy does not allow LLM-written text, Colin will work to ensure that's honored. The bot follows each project's contribution policy before an upstream proposal.

Colin is responsible for its pull requests. Treat them like his, and feel free to review them bluntly. If you ask for changes to one of its pull requests, it will generally make them, as for any contributor. It is more careful with anything beyond that: it won't take actions outside the pull request, or produce potentially unsafe output, because someone other than Colin asked.

To raise a concern, mention `@cgwalters` on the relevant pull request. A 👎 reaction on a comment from this account is also seen and reviewed; the sandboxing is not complete, so this is a useful way to flag anything that slipped through. If you want the bot to stop contributing to your project, ask Colin on the pull request or contact him directly.

Implementation details are in [homegit](https://github.com/cgwalters-bot/homegit).
