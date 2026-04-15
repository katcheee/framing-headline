# What Headlines Leave Out

An interactive article that guides a reader through three layers of context behind a single headline — the human story, the systemic context, and the data — with a Gemini-powered chat companion that opens each stage with a prompt and stays available for the reader to dig deeper.

**Live headline:** *Mayor Lurie says he's trading S.F. shelter beds for sites with more services. Critics aren't convinced.* (San Francisco Chronicle, April 2026)

## What this is

Most people form an opinion on homelessness from headlines. Headlines are necessarily short. This project takes one real headline and walks the reader through what that headline leaves out — not by lecturing, but by layering context on top of the same sentence.

The reader clicks into the article, sees the original summary, and is then asked by a conversational guide what their gut reaction is. From there, the reader moves through three layers — human, systemic, data — and ends back on the original headline. The LLM is present the whole way, opening each stage with a pointed question and available to be pushed on anything.

## How the LLM fits

The chat panel is a persistent companion that travels with the reader through the experience. It is not a quiz, a chatbot, or a fact-lookup. It is a guide built on a specific persona and a specific editorial stance:

- **Lead with humanity, follow with evidence.** Never open with a statistic where a human frame would land better.
- **Expand, don't correct.** When a reader brings in something they've heard, acknowledge what the source got right before complicating it.
- **Hold complexity without collapsing it.** Homelessness is not one problem. The guide is comfortable saying "it depends."
- **Invite, don't instruct.** Socratic prompts, but only when the reader is ready for them — not as a reflex.

The full system prompt lives in [`netlify/functions/chat.js`](netlify/functions/chat.js). It was iterated on through five golden-set prompt tests to identify gaps in how the LLM was handling empathy, framing, and the refusal to fall back on statistics too early.

### Stage openers

Each stage of the journey triggers a specific opening question from the guide. The full conversation carries across stages so earlier responses inform later ones.

| Stage | Opener |
|---|---|
| Focused headline | "What's your gut reaction to this headline?" |
| Layer 1 — Human Story | "Something here will probably land harder than the rest. What is it for you?" |
| Layer 2 — Systemic Context | "What's starting to connect — or what still feels off?" |
| Layer 3 — Data | "Any of these shift how you read the original headline?" |
| Return to headline | "What reads differently now than it did the first time?" |

## Architecture

- **Frontend:** single-file `index.html` — all HTML, CSS, and JS inline. No build step.
- **LLM proxy:** `netlify/functions/chat.js` is a Netlify serverless function that holds the system prompt and the `GOOGLE_API_KEY` (from Google AI Studio), so the key is never exposed to the browser.
- **Model:** `gemini-2.5-flash` — on the free tier, fast enough for an interactive demo.
- **Config:** `netlify.toml` points Netlify at the functions folder and publishes the repo root as the site.

Request flow:
```
browser  →  POST /.netlify/functions/chat  →  Gemini API  →  response back to browser
```

The browser sends the full conversation history plus a short `layerContext` string describing which stage the reader is currently on. The serverless function appends that context to the system prompt and translates the message format into Gemini's expected shape.

## Deploying

1. Push this repo to GitHub.
2. In Netlify, "Add new site" → "Import an existing project" → connect the GitHub repo.
3. Under **Site settings → Environment variables**, add:
   - `GOOGLE_API_KEY` = your Google AI Studio API key
4. Trigger a redeploy after adding the key.

That's it. `netlify.toml` tells Netlify everything else it needs.

## Running locally

You can open `index.html` directly in a browser, but the chat will fail because the Netlify function isn't running. To test the full flow locally:

```bash
npm install -g netlify-cli
netlify dev
```

`netlify dev` spins up both the static site and the serverless function at `http://localhost:8888`. Set `GOOGLE_API_KEY` in a `.env` file in the repo root or export it in your shell before running.

## Repo layout

```
.
├── index.html                    # the whole experience
├── netlify.toml                  # Netlify config
├── netlify/functions/chat.js     # LLM proxy + system prompt
├── Images/shelter.png            # article image
└── README.md
```

## Credits

Headline and source reporting: *San Francisco Chronicle.* This is a student-built, non-commercial demonstration of how editorial framing, context, and AI guidance can change how a reader holds a story.
