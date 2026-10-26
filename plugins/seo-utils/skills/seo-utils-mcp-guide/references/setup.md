# Setting up SEO Utils

SEO Utils serves its setup guide live: call `get_setup_guide` first (with
`skill_version`, see SKILL.md) and follow it. It always matches the user's
version of the app, so it wins over anything here. These rules apply
throughout.

## Rules

1. **Secrets go in the app, never in the chat.** API keys, passwords, the
   license key and key files are entered in SEO Utils. Open the right settings
   card with `open_setup_step`; the user pastes the value there and clicks
   Save. No setup action takes a secret. If the user pastes one in the chat
   anyway, tell them not to and ask them to paste it in the app.
2. **Verify each step.** When the user says they saved something, run
   `get_setup_status` again and confirm the step is `ready` before you go on.
   Never report a step as done from the user's word alone.
3. **Essentials first.** Ask what the user does, set up only what their tools
   need now, one step at a time. Offer optional steps only when they want
   them, and say they can ask you again any time.
4. **Say what it costs** before the user spends money: each step's
   `instructions` says. DataForSEO's minimum top-up is $50; for $10–20 of
   credit, mention the rented DataForSEO key.
5. **Finish with a first win.** When the needed steps are ready, open the
   first tool the user wants. With the MCP add-on (`full_mcp` is `available`
   in the status), offer to create something small with them, such as a rank
   tracker with five keywords.

## When something goes wrong

- `get_setup_guide` or another tool says SEO Utils isn't running: ask the user
  to open SEO Utils and turn on Settings → MCP Server, then try again.
- An action other than the setup ones answers with the purchase message: the
  user has setup-only access. Setup still works; the other tools need the MCP
  add-on.
- The guide says this skill is older or newer than the app: tell the user what
  it says to do, then go on with the guide.
