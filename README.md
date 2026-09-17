# Mesh Landing Page

Static marketing and showcase website for **[Mesh](https://github.com/VUXXE/Mesh)** — the open-source real-time vector whiteboard on Cloudflare Workers and Svelte 5.

## Tech Stack

- **Framework**: [SvelteKit](https://kit.svelte.dev/) (Svelte 5 Runes)
- **Styling**: [TailwindCSS v4](https://tailwindcss.com/)
- **Adapter**: `@sveltejs/adapter-static` (100% static HTML/CSS/JS export)
- **Deployment Target**: Cloudflare Pages, GitHub Pages, Vercel, or Netlify

## Getting Started

```bash
# Install dependencies
bun install

# Start local dev server
bun run dev

# Build static output for deployment (outputs to ./build)
bun run build
```

## Configuration

Set the target Mesh app instance URL in `.env`:

```env
VITE_APP_URL=https://mesh.asy.web.id
```

## License

MIT © [VUXXE](https://github.com/VUXXE)
