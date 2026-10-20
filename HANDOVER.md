# Handover: publish the Targum onboarding site

## What this is
Two static, self-contained HTML pages (no build step, no dependencies beyond Google Fonts):

- `index.html` — **Targum Setup Guide**. Five-question assessment that routes a visitor to one of three setup paths (Claude connector, ChatGPT developer mode, editor-only), with animated SVG mock screens, a copyable server URL (`https://targum.video/mcp`) and a first prompt. EN/ES.
- `challenges.html` — **Targum First Week**. Six guided challenges for agentic-first editing, progress saved in localStorage, a feedback form at the end. EN/ES. Links to `./index.html`.
- `targum-setup-guide.html`, `targum-challenges.html` — the same pages without the `<html>/<head>/<body>` wrapper (source for the Claude artifact previews). Not needed for hosting; keep or delete.
- `.nojekyll` — so GitHub Pages serves files as-is.

## Task
1. Push this folder to `https://github.com/francescogalletta/targum-onboarding` (branch `main`). A commit already exists locally; if you start from a fresh checkout just add the files and commit.
2. Enable GitHub Pages: Settings → Pages → Source: **Deploy from a branch**, Branch: `main`, folder `/ (root)`. Or via API:
   `gh api -X POST repos/francescogalletta/targum-onboarding/pages -f source[branch]=main -f source[path]=/`
3. Verify `https://francescogalletta.github.io/targum-onboarding/` and `/challenges.html` load, the EN/ES toggle works, and the result page renders after answering the five questions.
4. Optional: custom domain (e.g. `start.targum.video`) via a CNAME file + DNS CNAME to `francescogalletta.github.io`.

## Analytics / data (already live, nothing to deploy)
Both pages POST anonymously to Supabase project **targum-onboarding** (`bvqdeilzhvepczhpkkwe`, eu-west-1):
- `POST /rest/v1/rpc/upsert_onboarding_session` with `{p:{session_id, lang, mcp_familiarity, assistant, plan, video_editing, content_type, path, completed, user_agent, referrer}}` — one row per visitor in `onboarding_sessions`.
- `POST /rest/v1/onboarding_events` with `{session_id, event, step, value, lang, path}` — events: `page_view, quiz_start, answer, path_shown, signin_click, editor_click, step_link_click, copy, lang_change, restart, challenge_*`.
- Config lives at the top of each page's script in `CONFIG` (URL + publishable key; the key is public by design, RLS allows insert only).
- `onboarding_funnel` view gives quick counts. One stray test row `session_id='probe-check'` can be deleted.

## Known follow-ups (not required for publishing)
- Mock screens in the setup guide are illustrations; real screenshots of the Targum editor are planned. Each illustration is a `SCREENS.<name>()` function; swap for an `<img>`.
- The challenges page guesses some editor wording (SRT export button, "Select" vs "Approve" on a clip, where credits are bought). Fix once verified in the editor.
- Spanish is Spain-flavoured (vídeo, móvil).
- When the final URL is known, update `CONFIG.setupGuideUrl` in `challenges.html` only if the pages are not hosted side by side.
