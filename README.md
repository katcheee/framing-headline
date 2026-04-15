# What Headlines Leave Out

An interactive article that walks a reader through three layers of context behind a single headline — the human story, the systemic context, and the data — with a Gemini-powered narrator that carries them through the arc, listens, and leaves them with a path forward.

**Live headline:** *Mayor Lurie says he's trading S.F. shelter beds for sites with more services. Critics aren't convinced.* (San Francisco Chronicle, April 2026)

## What this is

Most people form an opinion on homelessness from headlines. Headlines are necessarily short. This project takes one real headline and walks the reader through what that headline leaves out — not by lecturing, but by layering context on top of the same sentence.

The reader clicks the headline, sees the summary, and is pulled into a five-beat journey: a focused first read, the human story, the systemic context, the data, and then a return to the original headline with action.

## The five-beat arc

1. **FOCUSED** — Reader sees the original headline and summary. No context yet.
2. **LAYER 1 — The Human Story** — The number 280 becomes people.
3. **LAYER 2 — The Systemic Context** — "More services" stops being a simple upgrade.
4. **LAYER 3 — The Data** — Rent vs. wages. The math of the closure.
5. **RETURN** — Same headline, different reader. Three curated paths forward appear below it.

## How the LLM fits

The LLM is not a chatbot sitting beside the story. It is the narrator walking through it.

Every stage follows a three-beat rhythm:

- **OPEN** — The narrator delivers a pre-scripted opener that sets the scene and ends in a specific, earned invitation. Generic "what do you think?" is banned. The Continue button is hidden until the opener lands so the reader actually reads it.
- **EXCHANGE** — The reader responds. The LLM replies in a structured way: react specifically to what they said (one line, no praise reflex), add exactly one thing they don't yet have, leave a thread — either a pointed invite to go deeper or a signal the stage is closing.
- **CLOSE** — When the reader signals readiness (with words like "yes," "I'm ready," "next"), the narrator delivers a two-line hand-off — one line synthesizing the stage, one line foreshadowing the next — and appends a `[[ADVANCE]]` token. The frontend strips the token and transitions the page automatically. The reader can also click Continue themselves; the LLM is an option, not a gate.

The LLM is also the **connective tissue across layers**. If the reader says something striking on Layer 1, the narrator is instructed to bring it back on Layer 3 or the return. That's what makes the three otherwise-disconnected panels feel like a single argument.

## Persona and editorial stance

The LLM is built on a specific persona developed across five golden-set prompt iterations:

- **Lead with humanity, follow with evidence.** Never open with a statistic where a human frame would land better.
- **Expand, don't correct.** When a reader brings in a misconception, acknowledge what the source got right before complicating it.
- **Hold complexity without collapsing it.** "It depends" is an allowed answer.
- **Invite, don't instruct.** No shame, no guilt, no moralizing. The Socratic method gets used only when the reader is ready.
- **Short sentences. Documentary voice, not therapy voice.**

The full system prompt lives in [`netlify/functions/chat.js`](netlify/functions/chat.js).

## Stage openers

Each stage opens with a scripted narrator beat. The full conversation history carries across stages so the LLM can reference what the reader said earlier.

| Stage | Opener |
|---|---|
| Focused | Read the headline and send the first word that comes up. |
| Layer 1 | Before the system, before the data: the 280 is not a number. It's a line of people who stood somewhere at 6 a.m. hoping to be seen. |
| Layer 2 | Now zoom out. This is where the fault line shows up. |
| Layer 3 | Numbers. Watch the last one — $3,000 rent against a full-time minimum-wage paycheck of $3,400. |
| Return | Same headline you started with. Read it one more time. What's different? |

## The exit ramp

Ending on *"the math doesn't work"* without an exit would leave the reader with weight and no agency. The return stage carries two pieces of action:

**A static "Ways in" panel** appears below the final headline with three curated next steps for this specific article:

- **Stand with** — Coalition on Homelessness SF (advocacy on SF shelter policy)
- **Show up** — GLIDE (meals, harm reduction, direct services in the Tenderloin)
- **Speak up** — SF Board of Supervisors (contact your supervisor about the 711 Post closure)

**The LLM tailors the same three**, not as a list dump but as one path. On the second or third exchange of the return stage, once the narrator can read which direction the reader is pulled, it names exactly one org and explains why it fits them specifically. Paralyzed readers get GLIDE by default — "show up with a meal" is a thing a body can do this week.

*Note: these actions are currently hardcoded for the 711 Post story. The panel is structured so that when the archive eventually loads dynamic articles, each article will carry its own `actions` array and the panel will render from that.*

## Architecture

- **Frontend:** single-file `index.html` — all HTML, CSS, and JS inline. No build step.
- **LLM proxy:** `netlify/functions/chat.js` is a Netlify serverless function that holds the system prompt and the `GOOGLE_API_KEY` (from Google AI Studio), so the key is never exposed to the browser.
- **Model:** `gemini-2.5-flash`. Thinking tokens are disabled since this is a short-narrative use case and the thinking budget was eating the visible output window.
- **Config:** `netlify.toml` points Netlify at the functions folder and publishes the repo root as the site.

Request flow:
```
browser  →  POST /.netlify/functions/chat  →  Gemini API  →  response back to browser
```

The browser sends the full conversation history plus a `layerContext` string describing which beat of the arc the reader is currently on. The serverless function appends that context to the system prompt and translates the message format into Gemini's expected shape.

## Deploying

1. Push this repo to GitHub.
2. In Netlify, **Add new site → Import an existing project** → connect the GitHub repo.
3. Under **Site configuration → Environment variables**, add:
   - `GOOGLE_API_KEY` = your Google AI Studio API key
4. Trigger a redeploy after adding the key so the function picks it up.

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

## Future work

- **Dynamic archive.** The archive sidebar currently holds placeholder entries. Eventually it should pull live articles from a news API, with each article carrying its own layer content, system prompt context, and action panel.
- **Per-article actions.** When the archive goes live, the three "Ways in" cards need to become article-specific rather than hardcoded to the 711 Post story.
- **Conversation persistence.** Right now the chat history resets when the reader returns home. A memory layer could let returning readers resume mid-arc or see their past reactions.

## Credits

Headline and source reporting: *San Francisco Chronicle.* This is a student-built, non-commercial demonstration of how editorial framing, context, and a narrator-style LLM can change how a reader holds a story — and where they can take it.
