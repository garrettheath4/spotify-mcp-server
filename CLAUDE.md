# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spotify MCP Server is a Model Context Protocol server that enables AI assistants (Claude, Cursor, VSCode/Cline) to control Spotify playback and manage playlists. Published as `@garrettheath4/spotify-mcp-server`.

## Commands

```bash
npm run build       # Compile TypeScript to build/ directory
npm run lint        # Run Biome linter (errors on warnings)
npm run lint:fix    # Auto-fix lint issues and organize imports
npm run typecheck   # TypeScript type checking without emit
npm run auth        # Run OAuth2 authentication flow with Spotify
```

## Architecture

The server uses the MCP SDK with Stdio transport. Tools are organized into three modules by operation type:

```
src/
├── index.ts     # MCP server entry point, registers all tools
├── auth.ts      # OAuth2 CLI tool for Spotify authentication
├── types.ts     # TypeScript types and tool interface definition
├── utils.ts     # Spotify API initialization, token management, helpers
├── read.ts      # 8 read-only tools (search, now playing, playlists, queue, etc.)
├── play.ts      # 10 playback/creation tools (play, pause, skip, create playlist, etc.)
└── albums.ts    # 4 album management tools
```

**Key patterns:**
- Each tool module exports an array of `tool<T>` definitions with name, description, Zod schema, and handler
- All tools are imported and registered in `index.ts` using `server.tool()`
- `utils.ts` provides `createSpotifyApi()` (singleton), `handleSpotifyRequest()` (error wrapper), and OAuth flow
- Tokens stored at `~/.config/spotify-mcp-server/tokens.json`

**Environment variables required:**
- `SPOTIFY_CLIENT_ID`
- `SPOTIFY_CLIENT_SECRET`
- `SPOTIFY_REDIRECT_URI` (default: `http://127.0.0.1:8888/callback`)

## Adding New Tools

1. Create tool definition following the `tool<T>` interface in `types.ts`
2. Add to appropriate module (`read.ts`, `play.ts`, or `albums.ts`)
3. Export from the module's array
4. Tool will be auto-registered via the spread in `index.ts`
