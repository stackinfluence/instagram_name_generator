# Stack Influence - Instagram Name Generator

A free, embeddable Instagram username generator. Enter a keyword or name, pick a
content category, add optional refinements and a creator type, and get ten
Instagram-ready ideas. Each idea is a @handle plus a matching display name, with
a copy button and a "check on Instagram" link.

This is a **Type B1** tool (AI-powered). It calls the Anthropic API from a
Netlify serverless function so the API key stays server-side.

## Files

- `index.html` - the tool (single static file, 100% ASCII)
- `netlify/functions/generate.js` - serverless function; holds the key, calls the API, returns JSON
- `netlify.toml` - publish + functions config
- `webflow-embed-snippet.html` - iframe to paste into Webflow (swap in the real URL after deploy)
- `webflow-cms-copy.md` - copy for every Webflow CMS field

Keep the two Webflow files OUT of the GitHub repo.

## Deploy (Netlify)

1. Push index.html, netlify.toml, README.md, and the netlify/ folder to GitHub.
   Keep the netlify/functions/ structure intact - if generate.js lands at repo
   root the function 404s. Confirm the repo shows netlify / functions / generate.js.
2. In Netlify, connect the repo. Build settings come from netlify.toml
   (publish ".", functions "netlify/functions"). No build command needed.
3. **Set the API key**: Site configuration -> Environment variables ->
   add ANTHROPIC_API_KEY (mark it secret). All scopes, same value for all deploy
   contexts. The function reads process.env.ANTHROPIC_API_KEY.
4. **Redeploy** so the function picks up the key: Deploys -> Trigger deploy ->
   Deploy site. (Env vars only apply on a fresh deploy.)
5. Confirm it generates on the Netlify URL, then paste that URL into the Webflow
   embed snippet (both the src and the origin check) and embed it.

Note: Netlify's free plan blocks build triggers fired from API pushes, but
normal GitHub-connected deploys and manual deploys work fine.

## Debugging a failed generate

If Generate shows an error, open DevTools -> Network -> click the failed
"generate" request -> Response. The function surfaces the real upstream message
in a `detail` field:
- "API key is invalid" -> the ANTHROPIC_API_KEY value is wrong or was revoked.
- "credit balance is too low" / 402 -> the key's account needs credit.
- "model not found" -> the MODEL string is not available to that account.

## How it works

- The page posts { keyword, category, refine, creatorType } to the function.
  keyword and category are required; refine is an array of up to two free-text
  extras (a sub-niche, vibe, or theme); creatorType is an optional identity
  (UGC creator, content creator, influencer, brand/business) that the model may
  weave into a few handles where it reads naturally.
- The function prompts the model (claude-sonnet-4-6) for exactly 10 ideas as a
  JSON array of { username, name, note }, strips code fences, parses, and
  **sanitizes each result**: the username is forced to Instagram's allowed set
  (lowercase letters, numbers, ".", "_"; 3-30 chars; no leading/trailing or
  doubled periods) and the display name is stripped to plain ASCII text (no
  emoji/symbols, max 30 chars). Anything whose username fails is dropped.
- The page renders each idea as a display name over an @handle, with a Copy
  handle button and a link to instagram.com/<username>/ so the user can check
  availability themselves. Instagram has no public availability API, which the
  disclaimer states plainly.

## Auto-resize

The tool posts its height to the parent on every content change using the unique
key `siIgNameHeight` (ResizeObserver on document.body plus a post on load). The
Webflow embed snippet listens for that key and resizes the iframe. The key is
unique so multiple tool embeds on one page do not collide.

## Tuning later

- **Categories**: edit the CATEGORIES array in index.html (id, label, emoji as
  \u escapes) and the matching CATEGORY_GUIDANCE map in generate.js. Keep the two
  in sync by id.
- **Creator types**: edit the CREATOR_TYPES array in index.html. The first entry
  (empty id, "No preference") is the default reset and should stay first.
- **Refine inputs**: the two optional free-text fields are combined into the
  refine array in index.html and shape some ideas without being forced into every
  handle (see refineLine in generate.js).
- **Username / name rules**: sanitizeUsername() and sanitizeName() in generate.js
  are the single source of truth. Adjust there if Instagram's rules change.
- **Model**: MODEL constant at the top of generate.js.

## ASCII check

    python3 -c "d=open('index.html','rb').read(); print('non-ASCII bytes:', len([x for x in d if x>127]))"

Must print 0 for both index.html and generate.js.
