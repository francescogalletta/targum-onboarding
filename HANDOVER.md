# Handover: publish the Targum onboarding site

## What this is
One static, self-contained HTML page plus a redirect (no build step, no dependencies beyond Google Fonts):

- `index.html` — **Targum Setup Guide**. Five-question assessment that routes a visitor to one of three setup paths (Claude connector, ChatGPT developer mode, editor-only), with a clickable link on every step that leaves the page, a copyable server URL (`https://targum.video/mcp`) and a first prompt. EN/ES.
- The same page continues into **Targum First Week** after the setup path (`#challenges`): six guided challenges tailored to the visitor's path (assistant prompts for Claude or ChatGPT, editor-only steps otherwise) and content type, progress saved in localStorage, a feedback form at the end. `challenges.html` is now a redirect to `index.html#challenges` for old links.
- `targum-setup-guide.html` — the same page without the `<html>/<head>/<body>` wrapper (source for the Claude artifact preview). Not needed for hosting; keep or delete.
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
- `POST /rest/v1/onboarding_events` with `{session_id, event, step, value, lang, path}` — events: `page_view, quiz_start, answer, path_shown, signin_click, editor_click, step_link_click, approval_open, copy, lang_change, restart, challenge_*`.
- Config lives at the top of each page's script in `CONFIG` (URL + publishable key; the key is public by design, RLS allows insert only).
- `onboarding_funnel` view gives quick counts. One stray test row `session_id='probe-check'` can be deleted.

## Known follow-ups (not required for publishing)
- The editor-only variants of the challenges (shown when the visitor has no assistant) describe editor actions in general terms (Search transcript, Clean up, Make clip, Render, SRT export). Verify the wording against the editor.
- The setup guide has no screenshots: the animated mock screens were removed because they were not exact renders of the real UIs. Real screenshots can be added per step by appending an `<img>` to the step body in `stepEl()`.
- The challenges page guesses some editor wording (SRT export button, "Select" vs "Approve" on a clip, where credits are bought). Fix once verified in the editor.
- Spanish is Spain-flavoured (vídeo, móvil).
