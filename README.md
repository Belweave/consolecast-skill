# Console Cast skill for coding agents

An [Agent Skill](https://agentskills.io) that teaches a coding agent (Claude Code, Codex, Cursor and others) to set up and run [Console Cast](https://consolecast.app) for you with the `consolecast` command: install it, check its status, connect a PS5 or Xbox Series X|S to OBS Studio without a capture card, add destinations, go live and stop, diagnose problems, update and uninstall. The agent asks before it changes anything, never handles your stream keys or license key, and never goes live without your yes.

## Install

```sh
npx skills add Belweave/consolecast-skill
```

Offline, the same skill comes with Console Cast itself:

```sh
consolecast --skill
```

prints it to standard output (save it as `SKILL.md` wherever your agent reads skills).

The skill lives in [`skills/consolecast/SKILL.md`](skills/consolecast/SKILL.md). It is generated from the Console Cast source, so changes here are made there; issues and questions: info@belweave.com.

## License

Apache-2.0, copyright Belweave. See [LICENSE](LICENSE).

PlayStation® and PS5® are registered trademarks of Sony Interactive Entertainment Inc. Xbox® is a registered trademark of Microsoft Corporation. OBS Studio™ is a trademark of the OBS Project. Console Cast is not affiliated with, endorsed by or sponsored by any of them.
