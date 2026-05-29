# Brief Builder

AI-powered campaign brief generator. Runs as a single HTML file hosted on GitHub Pages.

**Live site:** https://athrukshna.github.io/brief-builder

---

## Stack

- `index.html` — the entire app, no build step, no dependencies
- Cloudflare Workers — proxy that keeps the Anthropic API key secret
- GitHub Pages — free static hosting

---

## Setup

### 1. Cloudflare Worker

1. Go to [workers.cloudflare.com](https://workers.cloudflare.com) and create a new Worker
2. Paste this code:

```js
export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }
    try {
      const body = await request.json();
      const res = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': env.ANTHROPIC_API_KEY,
          'anthropic-version': '2023-06-01',
        },
        body: JSON.stringify(body),
      });
      const data = await res.json();
      return new Response(JSON.stringify(data), {
        status: res.status,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        }
      });
    } catch (err) {
      return new Response(JSON.stringify({ error: err.message }), {
        status: 500,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        }
      });
    }
  }
}
```

3. Go to **Settings → Variables and Secrets → Add**
   - Type: `Secret`
   - Name: `ANTHROPIC_API_KEY`
   - Value: your key from [console.anthropic.com](https://console.anthropic.com)
4. Deploy — your Worker URL will be `https://briefbuilder.athrukshna.workers.dev`

### 2. GitHub Pages

1. Push `index.html` to the `main` branch
2. Go to **Settings → Pages → Source → Deploy from branch → main / root**
3. Site is live at `https://athrukshna.github.io/brief-builder`

### 3. Connect in the app

1. Open the live site
2. Click **API Setup** (top right)
3. Paste your Worker URL
4. Click **Test connection** → green = ready
5. Click **Save & close**

---

## Repo structure

```
/
├── index.html       ← entire app
└── README.md
```
