# Jig plugins

Install Jig in **Claude** or **ChatGPT / Codex**, then make Instagram carousels, reels, video ads and narrated shorts just by asking.

One plugin installs both pieces:

- **Tools** — Jig's connector (feed drafts, image / video / voice generation, layered editing, rendering)
- **Skills** — production know-how for carousels, card news, reels, issue shorts and video ads

You sign in with your [Jig](https://jig.ai.kr) account during install.

## Claude (Pro, Max, Team, Enterprise)

1. Open **Customize** in the left sidebar, then the **Plugins** tab.
2. Next to **Personal plugins**, click **+** → **Add marketplace**.
3. Paste `https://github.com/smp2103/jig-plugins`.
4. Click **Install** next to **Jig** and sign in with your Jig account.

On the Free plan, plugins aren't available. Add the connector only: **Customize → Connectors → + → Add custom connector**, and paste the connector URL shown at [jig.ai.kr/welcome](https://jig.ai.kr/welcome).

## ChatGPT desktop app (Codex)

1. Open **Plugins** in the left sidebar (not the one inside Settings).
2. Top right **Add** → **Add marketplace**, paste `https://github.com/smp2103/jig-plugins`.
3. **Browse directory** → **Jig** → **Install (+)**.
4. Sign in with your Jig account in the browser window that opens, then start a new thread.

Leave the URL, token and header fields in the plugin details empty. Putting a value in the token field turns sign-in off.

## Codex CLI

```bash
codex plugin marketplace add smp2103/jig-plugins
```

Then open `/plugins` in Codex, install **Jig**, sign in, and start a new session.
The VS Code / IDE extension doesn't support plugins.

## Try it

Start a new chat and send one of these:

- "Make a 6-slide carousel comparing three iced coffees."
- "Turn this topic into a 45-second narrated short: why Google Maps doesn't work in Seoul."
- "Make a 15-second vertical ad for this product, with subtitles."

Finished posts show up at [jig.ai.kr](https://jig.ai.kr).

## Skills

| Skill | For |
| --- | --- |
| `feed-carousel` | Instagram feed posts and carousels |
| `mascot-cardnews` | Korean-style card news with a mascot |
| `story-format` | Narrated story videos from multiple clips |
| `variety-caption-style` | Korean variety-show caption styling |
| `ad-creative-core` | Shared base for layering video ads |
| `ad-performance` | Click and purchase-focused ads |
| `ad-brand` | Brand and awareness ads |
| `design-corpus` | Reusing saved design pieces |

In Codex you can call one directly, e.g. `$feed-carousel`. Otherwise just describe what you want.

## Updating

When a new version is out, reinstall **Jig** from the plugin screen. It applies to new threads.
