# FastDex

A fast, modern Pokédex web app. Search 1,000+ Pokémon, analyze matchups, build teams, track Nuzlocke runs, calculate damage, and hunt your living shiny dex — all from a single HTML page.

**Live at [fastdex.app](https://fastdex.app)**

---

## Features

- **Pokédex** — Stats, abilities, type matchups, moves with filters, evolutions, breeding info, flavor text, wild encounter locations, and held-item drop rates.
- **Team Builder** — Draft up to 6 Pokémon, see shared weaknesses, combined STAB coverage, suggested partners, and a graded team score.
- **Compare** — Stack two Pokémon side-by-side: stats, types, head-to-head matchup math.
- **Type Search** — Find every Pokémon of a given type or dual-type combination.
- **Catch 'Em All** — Track your living dex (regular + shiny) with per-generation progress bars and stats. Import/export JSON.
- **Nuzlocke Mode** — Run tracker with party/box/graveyard, encounter logging, level caps, rules enforcement, gym leader spoilers, and multi-run support. Covers every mainline game from Red/Blue to Scarlet/Violet.
- **Damage Calculator** — Full Gen 6+ damage formula with natures, IVs/EVs, stat stages, abilities, items, weather, terrain, screens, and crits. Imports Pokémon directly from your Nuzlocke runs.

All data is cached aggressively in `localStorage` — after first load, the app works almost instantly.

---

## Architecture

FastDex is a **single static HTML file** (no build step, no framework, no backend code) backed by a self-hosted PokéAPI mirror. The stack:

- **App**: `index.html` served by [Cloudflare Pages](https://pages.cloudflare.com/) (free)
- **API mirror**: [Cloudflare Worker](https://workers.cloudflare.com/) in front of a [Cloudflare R2](https://www.cloudflare.com/developer-platform/r2/) bucket holding the full [PokéAPI static data dump](https://github.com/PokeAPI/api-data) (~500 MB, 14 k+ objects)
- **Data flow**: browser → `fastdex.app` (HTML) → `api.fastdex.app` (Worker) → R2 (JSON) → response cached at Cloudflare's edge forever (`Cache-Control: immutable`)

The Worker performs on-the-fly name → ID resolution so the app can use human-readable endpoints like `/api/v2/pokemon/charizard` against a dump that's only indexed by numeric ID.

This means **zero load on pokeapi.co** regardless of traffic, and the edge cache handles the vast majority of requests so the home R2 bucket barely sees any reads.

---

## Running Locally

The app is a single file — no build, no dependencies. Just open `index.html` in a browser, or serve it with any static file server:

```bash
# Using Python
python3 -m http.server 8080

# Using Node
npx serve .

# Using Docker + nginx
docker run -p 8080:80 -v $(pwd):/usr/share/nginx/html:ro nginx:alpine
```

Then visit `http://localhost:8080`.

By default the app points at `https://api.fastdex.app/api/v2`. If you want to point it at the public PokéAPI instead (or your own mirror), edit the `POKEAPI` constant near the top of the `<script>` tag in `index.html`.

---

## Self-Hosting Your Own Mirror

Want to run an independent stack? Rough outline:

1. Download the [PokéAPI static data dump](https://github.com/PokeAPI/api-data)
2. Upload the `data/api/` directory to any static host or object store (R2, S3, a local nginx, etc.)
3. Put a small reverse proxy in front that:
   - Rewrites `/api/v2/{resource}/{name}` → `/api/v2/{resource}/{id}/index.json` (needs a name→ID lookup from each resource's `index.json`)
   - Serves responses with long cache headers
4. Point the app at your new API origin

For a reference Worker implementation, see the one running at `api.fastdex.app`. It's about 60 lines of JavaScript.

---

## Credits

- **Data**: [PokéAPI](https://pokeapi.co/) — an open, comprehensive Pokémon REST API. FastDex uses their [static data dump](https://github.com/PokeAPI/api-data), self-hosted to avoid putting load on their servers.
- **Sprites**: Also from [PokéAPI](https://github.com/PokeAPI/sprites)
- **Type matchups**: Gen 6+ chart (Fairy included)
- **Nuzlocke level caps**: Based on community hardcore conventions

Pokémon and all associated names are trademarks of Nintendo / Game Freak / The Pokémon Company. FastDex is a non-commercial fan project.

---

## License

MIT — do whatever you want with it. PRs and issues welcome.
