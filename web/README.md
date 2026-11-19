# Local council frontend

[香港廣東話](README_zhHK.md)

This folder contains the dependency-free browser UI for the oMLX council. It can run
behind the local Python proxy or as a static GitHub Pages site that calls a reachable
oMLX server directly from the browser.

- `index.html` defines the three-seat workspace, human checkpoint, review controls, and
  chairman controls.
- `styles.css` contains the responsive visual system.
- `preferences.js` manages English, Hong Kong Cantonese, and appearance preferences.
- `app.js` discovers oMLX models, stores drafts/settings/history in localStorage, and
  either calls the local server's `/api/council`, `/api/council/review`, and
  `/api/council/synthesize` endpoints or the oMLX OpenAI-compatible `/v1` endpoints
  directly when a server address is configured.

Click **oMLX** in the top bar to set a server address and optional API key. Those
settings are kept in this browser's localStorage. A static HTTPS deployment must call
an HTTPS oMLX endpoint that permits the Pages origin through CORS; never expose an
unauthenticated inference server to the public Internet. See
[`../docs/omlx-local-council.md`](../docs/omlx-local-council.md) for installation,
Windows/GitHub Pages access, privacy, and user instructions.
