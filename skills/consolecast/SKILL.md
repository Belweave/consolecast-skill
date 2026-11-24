---
name: consolecast
description: Set up and run Console Cast for a person with the consolecast command. Console Cast brings a PS5 or Xbox Series X|S console's built-in Twitch broadcast into OBS Studio on their computer with no capture card, and sends OBS's output to several destinations. Use when someone asks to install, set up, check, fix, update or remove Console Cast, or to get their PS5 or Xbox gameplay into OBS or live through it.
license: Apache-2.0
metadata:
  publisher: Belweave
  homepage: https://consolecast.app
  contact: info@belweave.com
  version: "0.3.3"
---

# Console Cast

Console Cast runs on the person's computer. The console is pointed at the computer as its DNS server, so the console's own Twitch broadcast lands in Console Cast instead, and OBS Studio picks it up as a source. It works for 7 days with no account, card or key (the free week), then needs a subscription the person buys on https://consolecast.app/pricing.

You drive it with the `consolecast` command. `consolecast help` lists every command; `consolecast help NAME` explains one. Errors go to standard error, results to standard output.

## Rules: never break these

- **Ask before you change anything.** Installing, updating, uninstalling, starting, arming and disarming all need the person's yes first. Reading (status, diagnose, logs, setup values, version, `--dry-run`) needs none.
- **Never handle secrets.** Never ask for, read, print, store or pass on a stream key, a license key or the AI agent token. The person enters them in Console Cast's own window. So never run `consolecast license activate`, `consolecast output save`, `consolecast agents token` or `consolecast agents rotate-token`, and never open Console Cast's settings folder or its files.
- **Going live is public.** Never start a destination or OBS program output without the person's clear yes in the chat for that destination, at that moment. Never answer Console Cast's own questions for the person: do not pipe `y` into it, and do not add `--yes` to get past a broadcast that is on air.
- **The person changes the console, the router and the firewall.** Give them the exact values and menu paths; never change router or console settings yourself.
- **Passwords are typed by the person.** Commands that need administrator rights ask in their terminal (sudo, macOS, Windows prompts). If yours has no terminal, give the person the command to run.
- **Buying:** the person subscribes on https://consolecast.app/pricing. Never offer a coupon, discount or deal.
- **Not offered:** Console Cast Cloud and account sign-in. Do not offer or mention them, and do not run `consolecast cloud` or `consolecast account`.
- **Leave the AI Agents setting alone** (`consolecast agents on|off|level|live-without-asking`): what agents may do is the person's choice in the app.

## 1. Is it installed?

```sh
consolecast version --json
```

`{"version":"0.3.3","os":"darwin","arch":"arm64","install_kind":"mac-app","build":"release","self_update":true}`. `install_kind` is how it was installed: `mac-app`, `homebrew`, `deb`, `rpm`, `tarball`, `portable`, `windows`, or `unknown` (a copy no installer made; it cannot update or remove itself). Command not found: install it (section 2).

## 2. Install (ask first)

Needs: macOS 14 or later (Apple silicon or Intel); Linux x64 or ARM64 with glibc (Ubuntu 22.04+, Debian 12+, Fedora 40+, openSUSE, others through the tarball); OBS Studio 28 or later; the console and the computer on the same home network.

- macOS and Linux, one command (it checks every signature before installing, and asks for the person's password):
  `curl -fsSL https://consolecast.app/install.sh | bash`
  Show the plan first with `curl -fsSL https://consolecast.app/install.sh | bash -s -- --dry-run`.
- macOS with Homebrew: `brew install --cask belweave/tap/console-cast`
- Downloads (disk image for Mac; .deb, .rpm and tarball for Linux x64 and ARM64): https://consolecast.app/download
- Windows: coming very soon. When https://consolecast.app/download offers it, in PowerShell: `irm https://consolecast.app/install.ps1 | iex`

The install script, Homebrew and the Linux packages put the `consolecast` command on PATH. A Mac app dragged from the disk image does not: there the command is `"/Applications/Console Cast.app/Contents/Resources/consolecast"`. Run `consolecast version --json` to confirm.

## 3. Start and arm

Console Cast must be running for the console to stay online (the console uses it for DNS).

- With the app: open it (macOS `open -a "Console Cast"`; Linux `console-cast`; Windows the Start menu) and ask the person to click **Start Console Cast**. The first start begins the free week.
- Without the app (a server, a terminal): `consolecast up` runs in the foreground until Ctrl-C; start it in its own terminal or service.

Then read the state:

```sh
consolecast status
```

```
Console Cast 0.3.3: ARMED
  Console DNS: primary 192.168.1.20, secondary 0.0.0.0
  Console:     seen at 192.168.1.30 (4s ago)
  ...
```

First line: `ARMED` (ready), `LIVE` (the console's picture is arriving), `PASSTHROUGH` (the console streams straight to Twitch: run `consolecast arm` with the person's OK), `STARTING`, `ERROR` (see section 7), or `NOT RUNNING`. `consolecast status --json` gives the same as JSON (`state`, `stream.live`, `console.seen`, `obs.websocket_enabled`, `warnings`).

License: `consolecast license status` (JSON). `free_week.days_left` counts the free week; `active: false` with `required: true` means the week is over: the person subscribes on https://consolecast.app/pricing and pastes the key into the app's **Purchase license key** field.

## 4. Connect the console (the person does this)

```sh
consolecast setup ps5     # or: consolecast setup xbox
```

It prints the two DNS values and the menu path, for example `Primary DNS: 192.168.1.20` and `Secondary DNS: 0.0.0.0`. Give them to the person exactly:

- PS5: Settings → Network → Settings → Set Up Internet Connection → their connection → Advanced Settings → DNS Settings: Manual.
- Xbox Series X|S: Settings → General → Network settings → Advanced settings → DNS settings: Manual. Restart the Xbox. If it rejects the secondary, the same address goes in both slots (the output says so).

Then ask them to start the console's Twitch broadcast (PS5: Create → Broadcast → Twitch; Xbox: Capture & Share → Live streaming → Twitch). `consolecast status` should show `Console: seen at …` and then `LIVE`.

## 5. OBS

The person turns on OBS's WebSocket once: OBS → Tools → WebSocket Server Settings → Enable WebSocket server. Then, with OBS open:

```sh
consolecast obs            # Media Source; or: consolecast obs webrtc (lowest latency)
```

`ok: added to OBS scene "Scene"`. For a ready-made scene with an overlay on top: `consolecast obs-setup --overlay overlay.png`. `consolecast viewer` opens the live picture in a browser (for window capture in Discord and other apps).

## 6. Destinations, going live, stopping

- Saving a destination needs its stream key, so **the person saves it in the app**: Consoles & Destinations… → Local Outputs. Never paste or type a key yourself.
- See what is saved: `consolecast output list` (JSON: `id`, `host`, `state`; no keys). `state` is `streaming` when it is live, `retrying` when it keeps reconnecting.
- Going live, only after the person says yes for that destination now: say exactly what happens ("YouTube may go live as soon as OBS sends video"), then
  1. `consolecast tool obs_program_start` (OBS sends its program output to Console Cast; it asks in the terminal if a destination is already running),
  2. `consolecast output start ID`,
  3. `consolecast output list` until that destination is `streaming`. Never say they are live before that.
- Stop: `consolecast output stop ID`, then `consolecast tool obs_program_stop`.
- `consolecast disarm` sends a live console broadcast straight to Twitch, publicly: ask first. `consolecast down` quits Console Cast; the console then has no DNS until it runs again. When the person stops using Console Cast, they set the console's DNS back to Automatic (same menus as section 4).

Agents that speak MCP can also use `consolecast mcp`, Console Cast's own agent server: it applies the permission level the person chose in the app and asks them before going live.

## 7. Diagnose

Start from Console Cast's own diagnosis and repeat its fix; do not guess.

```sh
consolecast diagnose
```

A summary, then findings marked `[blocker]`, `[warning]` or `[info]`, each with its fix. Fix blockers first. More detail:

- `consolecast dns-test`: each test name `ok` or `FAIL`. All FAIL: Console Cast is not running, or a firewall blocks port 53.
- `consolecast logs 50 --tag err` (tags `cc`, `dns`, `obs`, `err`, `mtx`; `--grep TEXT` filters).
- `consolecast network`: this computer's IPv4 addresses (JSON); if the console used another one, give it the address shown by `consolecast setup ps5`.

Common findings: the console never used Console Cast's DNS (DNS not set, or the computer's address changed: give the values again); pass-through (`consolecast arm`); OBS WebSocket off (the person enables it, then `consolecast obs`); a destination retrying (wrong or expired key: the person saves it again in the app). On Linux the firewall can block the console: the person clicks **Allow Console Cast** in Consoles & Destinations… → Settings → Firewall. Never run `sudo` commands without showing them and getting a yes.

## 8. Update

```sh
consolecast update --dry-run    # downloads and checks everything, changes nothing
consolecast update              # with the person's OK
```

It installs the newest release the way Console Cast was installed, only after the signed release list, the file's SHA-256 and the operating system's own signature check all pass; any failure changes nothing. It asks once in the terminal; `--yes` skips that question, so add it only after the person said yes. While Console Cast is on air it stops with exit code 3 and says so: tell the person, and wait, or use `consolecast update --wait` (waits until the broadcast ends). Homebrew installs: it says to run `brew upgrade --cask console-cast` instead. `consolecast update --json` prints the result as JSON (`action`: `updated`, `up-to-date`, `verified`, `installing`, `use-homebrew`). `consolecast update check` only reports whether an update is available (JSON). A development build says it does not update itself: the person downloads a release.

## 9. Uninstall

```sh
consolecast uninstall --dry-run   # lists exactly what it would remove
consolecast uninstall             # with the person's OK; asks once
```

It removes the app, the engine, the `consolecast` command and what Console Cast set up (firewall rules, start at login), plus settings, saved destinations and keys. `--keep-settings` keeps settings and keys. It always keeps the record of the free week (reinstalling does not start another). Afterwards the person sets the console's DNS back to Automatic. The install script does the same: `curl -fsSL https://consolecast.app/install.sh | bash -s -- --uninstall`.

## Exit codes

| Code | Meaning | You |
| --- | --- | --- |
| 0 | Done (or nothing to do) | Report the result |
| 1 | Failed; for update and uninstall nothing was changed | Show the message; follow its advice |
| 2 | Wrong command line | Check `consolecast help NAME` |
| 3 | Not done: it needed a yes (on air, no terminal to ask, or the answer was no) | Ask the person; never add `--yes` on your own |

## More

Docs: https://consolecast.app/docs. Contact: info@belweave.com. Publisher: Belweave.

PlayStation® and PS5® are registered trademarks of Sony Interactive Entertainment Inc. Xbox® is a registered trademark of Microsoft Corporation. Twitch® is a registered trademark of Twitch Interactive, Inc. YouTube™ is a trademark of Google LLC. Discord® is a registered trademark of Discord Inc. OBS Studio™ is a trademark of the OBS Project. Console Cast is not affiliated with, endorsed by or sponsored by any of them.
