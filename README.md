# Infinity Nginx Reverse Proxy

This repository stores the Docker Compose definition and Nginx reverse-proxy configuration for the Infinity application domains.

## Repository layout

```text
infinity-proxy/
├── compose.yaml
├── .gitignore
├── README.md
└── conf/
    └── default.conf
```

> `conf/default.conf` must exist before you start the container. Add the Nginx configuration you have reviewed and tested. Do not commit passwords, tokens, SSH private keys, TLS private keys, or other secrets.

## Domain-to-application mapping

The following mappings were discussed for this deployment. Verify them against the current `conf/default.conf` and the services running on the target server before deploying.

| Domain | Backend port |
|---|---:|
| `loan.duruj.co` | `8180` |
| `admininfinity.duruj.co` | `3002` |
| `appinfinity.duruj.co` | `3001` |
| `infinitynotify.duruj.co` | `8182` |
| `wallet-admin.duruj.co` | `3004` |
| `wallet.duruj.co` | `7070` |

## Requirements

- Docker Engine and the Docker Compose plugin installed on the server.
- Git installed on the server.
- The repository accessible to the deploying administrator.
- DNS records and firewall rules pointing the relevant domains to the server.
- Backend applications running and reachable on the ports used in `conf/default.conf`.
- Any TLS certificates referenced by `default.conf` present at the configured paths on the server.

## First deployment

1. Clone the repository and enter the project directory:

   ```bash
   git clone <YOUR_GITHUB_REPOSITORY_URL>
   cd infinity-proxy
   ```

2. Review `compose.yaml` and `conf/default.conf`. Confirm domain names, backend ports, certificate paths, and proxy headers.

3. If the Nginx configuration references TLS certificates, ensure those certificates exist on the server and mount their directory read-only in `compose.yaml`. Do not put private keys in Git.

4. Test the configuration using the same image and mounted configuration:

   ```bash
   docker compose run --rm --no-deps --entrypoint nginx nginx -t
   ```

5. Start Nginx:

   ```bash
   docker compose up -d
   ```

6. Verify the container and configuration:

   ```bash
   docker ps
   docker exec infinity-nginx nginx -t
   docker logs --tail 100 infinity-nginx
   ```

## Updating an existing deployment

From the project directory:

```bash
git pull
docker compose run --rm --no-deps --entrypoint nginx nginx -t
docker exec infinity-nginx nginx -t
docker exec infinity-nginx nginx -s reload
```

Only reload after the configuration test succeeds. If the test fails, do not reload; correct the issue first. If `git pull` reports local changes or a conflict, inspect and resolve it before proceeding.

For changes to the Compose definition or container image, review the changes and use:

```bash
docker compose up -d
```

## Networking note

This Compose file uses Linux host networking. With `network_mode: host`, Nginx shares the host's network namespace, so Docker `ports:` mappings are not used. Confirm that ports 80 and 443 are available and that no other service is already listening on them.

## Certificate note

The Compose file includes an example, commented-out Let's Encrypt mount. Uncomment it only if the server stores certificates under `/etc/letsencrypt` and the paths in `default.conf` match. Otherwise, mount the actual certificate directory read-only.

## Security checklist

- Keep the repository private if it contains internal infrastructure details.
- Never commit private TLS keys, SSH private keys, credentials, tokens, or production `.env` files.
- Restrict server deployment access to authorised administrators.
- Test configuration changes before reloading Nginx.
- Keep a known-good version of the configuration available for rollback.
