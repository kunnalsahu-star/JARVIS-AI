# JARVIS AI Platform

JARVIS is a responsive, dependency-free static frontend with a separate Node.js backend for Gemini text conversations and image generation. Without a deployed backend, AI remains in an honest offline/demo state; it does not generate simulated responses or placeholder images.

## Frontend

Serve this directory with a static web server (for example, `npx http-server . -p 5500`) rather than opening the pages with `file://`. Pages include the home page, chat, dashboard, settings, profile, login, signup, checkout, about, and contact.

The existing chat history is stored in this browser's localStorage. It is not encrypted, cloud-synced, or guaranteed to survive clearing browser data. Voice input and speech synthesis depend on browser support.

## Gemini backend

The `backend` directory contains the Node.js API. Gemini credentials stay on that server and are read from `GEMINI_API_KEY`; never put a real key into frontend files. The API supports text chat and Gemini image generation. See [backend/README.md](./backend/README.md) for local setup and exact Render deployment steps.

To connect a deployed API, set `API_BASE_URL` in `js/config.js` to its HTTPS origin. It is deliberately blank until an actual Render service URL exists; the URL must not be guessed. The API uses Google's official `@google/genai` SDK and the documented stable `gemini-3.8-flash` text and `gemini-nano-banana-2.1` image models checked on 23 October 2026. Confirm model access, quota, and pricing for the configured Google AI Studio account before deployment.

## Hosting status

The frontend is static. The Node backend must run on a host that explicitly supports Node.js; no backend or `.htaccess` configuration is present in this project, and compatibility with the current InfinityFree account has not been proven. Do not claim that production AI is enabled until the API is deployed, configured, and tested.

## Project layout

```text
JARVIS/
├── backend/   Gemini API service and server-only environment configuration
├── css/       Shared, chat, dashboard, and responsive styles
├── js/        Frontend application, chat, and browser voice support
├── assets/    Images, icons, logos, and audio assets
└── *.html     Product pages
```
