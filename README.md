# Dev Container + Cloudflare Tunnel Boilerplate

Monorepo example:

- [**NestJS** API](https://github.com/unibrix/nestjs-boilerplate) (`backend` submodule, default port **4242** via `APP_PORT`)
- [**NextJS** web](https://github.com/unibrix/nextjs-boilerplate) (`frontend` submodule, port **3003**)
- **Postgres** (**5432**) + **Adminer** (**8080**) + **Mailcatcher UI** (**1080**) in [.devcontainer/docker-compose.yml](.devcontainer/docker-compose.yml)

## Quick start (VS Code)

1. Open repo in VS Code
2. `Cmd + Shift + P > Dev Containers: Reopen in Container`
3. Verify local endpoints:
   ```bash
   curl -I http://localhost:3003
   curl -I http://localhost:4242
   curl -I http://localhost:8080
   ```

## Cloudflare Tunnel

Tunnel runs as a separate `cloudflared` service in [.devcontainer/docker-compose.yml](.devcontainer/docker-compose.yml).
Set `CLOUDFLARED_TOKEN` in [.devcontainer/.env](.devcontainer/.env.example) to enable it.

See [docs/CONTAINERS-DEMO.md](docs/CONTAINERS-DEMO.md) for presentation script and rollout notes.
