# Dev Container Notes

## Services and ports

Defined in [.devcontainer/docker-compose.yml](docker-compose.yml):

- `frontend`: `http://localhost:3003`
- `backend`: `http://localhost:${APP_PORT}` (default `4242`, from `.devcontainer/.env`)
- `adminer`: `http://localhost:8080`
- `mailcatcher`: `http://localhost:1080` (SMTP `1025`)
- `db`: `localhost:5432`

## Useful commands

From repo root:

```bash
docker compose -f .devcontainer/docker-compose.yml ps
docker compose -f .devcontainer/docker-compose.yml logs -f backend
docker compose -f .devcontainer/docker-compose.yml logs -f frontend
```

`cloudflared` behavior:

- `cloudflared` service is part of compose.
- If [.devcontainer/.env](.env.example) has non-empty `CLOUDFLARED_TOKEN`, tunnel starts.
- If `CLOUDFLARED_TOKEN` is empty, `cloudflared` still starts but exits with an error, and no tunnel is created.
- After changing token value, run `Dev Containers: Rebuild and Reopen in Container`.

## GitHub commits from container (SSH agent forwarding)

On host machine add SSH key used for GitHub to keychain.

Example:
`ssh-add --apple-use-keychain ~/.ssh/id_ecdsa`

Check key in agent:
`ssh-add -l`

Verify GitHub auth:
`ssh -T git@github.com`

Expected output:

```text
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Then run:
`Dev Containers: Rebuild and Reopen in Container`
