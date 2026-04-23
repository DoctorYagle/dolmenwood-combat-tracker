# Dolmenwood Combat Tracker — Owlbear Rodeo Extension

A multiplayer combat tracker that walks your table through the Dolmenwood round sequence one phase at a time. Each phase displays a prompt and waits for every joined player to press Ready before the "Next Phase" button enables.

## Phase sequence

1. **Declarations** — spells, fairy runes, fleeing melee
2. **Initiative** — one party roll, Referee rolls for other sides
3. **Movement** — up to half Speed, full Speed if fleeing
4. **Missile Attacks** — ranged, possible at >5 ft
5. **Magic** — spells, runes, glamours, items, turn undead
6. **Melee Attacks** — melee, other actions, possible at ≤5 ft
7. **Morale** — Referee rolls for monsters/NPCs if applicable

After Morale, "End Round" bumps the round counter and resets to Declarations.

## Hosting

This is a pure static site with no build step. The simplest path is GitHub Pages:

1. Create a new GitHub repo, push these files to it (`manifest.json`, `icon.svg`, `index.html`, `README.md`).
2. In repo Settings → Pages, set source to `main` branch, root folder. Save.
3. GitHub gives you a URL like `https://yourname.github.io/yourrepo/`.
4. Your manifest URL is `https://yourname.github.io/yourrepo/manifest.json`.

Any static host works: Netlify, Vercel, Cloudflare Pages, an S3 bucket, your own server. Owlbear requires HTTPS.

## Installing in Owlbear Rodeo

1. Go to your Owlbear Rodeo profile.
2. Click **Add Extension** at the bottom.
3. Paste your manifest URL (ends in `/manifest.json`).
4. Create a room (or edit an existing one) and enable the extension.
5. Open the room. You'll see a sword icon in the top-left action bar. Click it to open the tracker.

## How it works

State is stored in Owlbear's shared room metadata, so every player sees the same round, phase, and ready statuses in real time. There's no server of your own — the hosted files are static; the live sync is handled by Owlbear.

Each player clicks **Join combat** to add themselves to the tracker. They press **Ready** when they've resolved their action for the current phase. Any player can click **Next Phase** once everyone joined has marked ready. Any player can click **Reset Combat** to clear everything back to Round 1, Declarations.

## Local development

Because it's plain HTML with a CDN import, you can just open `index.html` in a browser to see it (though the Owlbear SDK calls will error out without a parent Owlbear iframe). For real testing, host it somewhere reachable — even a local server works if you expose it via a tunnel like `ngrok` or `cloudflared`:

```bash
cd dolmenwood-combat-tracker
python3 -m http.server 8080
# in another terminal:
cloudflared tunnel --url http://localhost:8080
```

Then point Owlbear at the manifest URL on the tunnel.

## Customizing

- **Change phase prompts or add/remove phases:** edit the `PHASES` array near the top of `index.html`. Each entry has an `id` (unique, used as the key for per-phase ready state), `name` (shown in the header and prompt), and `prompt` (the instruction text).
- **Change colors:** edit the CSS custom properties in the `:root` block at the top of the `<style>` tag.
- **Change the icon:** replace `icon.svg`.

## Notes

- The extension doesn't track NPCs/monsters — it's for the PC-side sequencing you actually need coordinated at the table. The Referee handles NPC declarations and rolls offline, using the prompts as the shared clock.
- Players who leave the room mid-combat stay in the state object but drop off the visible list (you see only currently-connected joined players). They'll reappear if they reconnect.
- "Next Phase" is available to any player, not just the GM. If you want to lock this to GMs only, change the button's `disabled` logic in `render()` to also check `selfRole === "GM"`.
