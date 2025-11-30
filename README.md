# Duploflow Open WebUI Stack

Git-driven stack for deploying Open WebUI behind your existing Traefik + Cloudflare Tunnel with MCPO (MCP → OpenAPI bridge) alongside it. MCPO config lives in-repo so Portainer Git stack redeploys stay in sync.

## Repo Layout
- `docker-compose.yml` — Open WebUI + MCPO services, wired to the external `proxy` network and Traefik labels for Open WebUI.
- `.env.example` — environment variables to copy to `.env` or set in Portainer.
- `mcpo-config/config.json` — single source of truth for MCPO servers; bind-mounted into the container.

## Quickstart (Portainer Git Stack)
1. Ensure the external network exists: `docker network ls | grep proxy` (create with `docker network create proxy` if missing).
2. In Portainer → Stacks → Add stack → From Git repository:
   - Repository URL: `https://github.com/<your-user>/duploflow-openwebui-stack.git`
   - Compose path: `/docker-compose.yml`
   - Set environment variables (or point to `.env` in the repo).
3. Deploy. Traefik will route `https://${OPEN_WEBUI_HOST}` (e.g. `chat.duploflow.com`) to Open WebUI; MCPO stays internal.

## Environment
Copy `.env.example` to `.env` and fill real secrets:
- `OPEN_WEBUI_HOST` — public hostname (e.g. `chat.duploflow.com`).
- `WEBUI_SECRET_KEY` — long random secret for Open WebUI sessions.
- `OLLAMA_BASE_URL` — LLM backend (e.g. `http://ollama:11434`).
- `MCPO_API_KEY` — shared secret for MCPO; reuse in Open WebUI tool configs.
- Optional: `OPENAI_API_BASE_URL`, `OPENAI_API_KEY` for an OpenAI-compatible backend.

## Verification Checklist
After deploy:
- Containers up: `docker ps | grep open-webui` and `docker ps | grep mcpo`.
- On `proxy` network: `docker inspect open-webui | grep proxy` (same for `mcpo`).
- DNS/Tunnel: `chat.duploflow.com` proxied via Cloudflare tunnel to Traefik.
- Web: `https://chat.duploflow.com` loads Open WebUI; create admin user.
- Internal MCPO reachability (optional): `docker exec -it open-webui ping -c 1 mcpo`.

## Adding MCP Servers Later
Edit `mcpo-config/config.json` and commit:
```json
{
  "servers": [
    { "name": "time", "url": "http://time-mcp:3000", "api_key": "" }
  ]
}
```
Portainer Git redeploy + MCPO `--hot-reload` pick up changes automatically. In Open WebUI admin, register MCPO endpoints (`http://mcpo:8000/<tool-name>`) with header `x-api-key: ${MCPO_API_KEY}`.
