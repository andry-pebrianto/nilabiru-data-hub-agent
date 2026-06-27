# Nilabiru Portainer Agent

A lightweight Docker Compose setup that runs the Portainer Agent, enabling remote Docker environment management from a central Portainer CE instance in the Nilabiru ecosystem.

---

## Overview

**Nilabiru Portainer Agent** deploys a single [Portainer Agent](https://docs.portainer.io/admin/environments/add/docker/agent) container on a remote Docker host. Once running, the agent exposes port `9001` so that a central Portainer CE instance (such as the one running in `nilabiru-data-hub`) can connect to and manage this host's containers, images, volumes, and networks from a single web UI. Deployments are fully automated via GitHub Actions on every push to `main`.

---

## Services

| Service                      | Image                    | Port   | Description                                                                          |
| ---------------------------- | ------------------------ | ------ | ------------------------------------------------------------------------------------ |
| **nilabiru-portainer-agent** | `portainer/agent:2.42.0` | `9001` | Portainer Agent that exposes the local Docker environment to a Portainer CE instance |

---

## Requirements

- Docker Engine `20.10+`
- Docker Compose `v2+`
- Port `9001` reachable from the host running Portainer CE (either via Tailscale, VPN, or direct network access)
- A running Portainer CE instance to connect to this agent

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/andry-pebrianto/nilabiru-data-hub-agent.git
cd nilabiru-data-hub-agent
```

### 2. Start the agent

No environment variables or `.env` file are required. Simply run:

```bash
docker compose up -d
```

To verify the agent is running:

```bash
docker compose ps
```

### 3. Connect from Portainer CE

In your Portainer CE instance:

1. Go to **Environments → Add environment**.
2. Choose **Docker Standalone** → **Agent**.
3. Enter a name for this environment and set the **Agent URL** to `<HOST_IP>:9001`.
4. Click **Add environment**.

> **Note:** Replace `<HOST_IP>` with the IP address or hostname of the machine running this agent that is reachable from your Portainer CE instance (e.g. a Tailscale IP).

---

## Data Persistence

This service does not persist any data of its own. It mounts two paths from the host read-write so Portainer CE can inspect and manage them:

| Mount                     | Type       | Purpose                             |
| ------------------------- | ---------- | ----------------------------------- |
| `/var/run/docker.sock`    | Bind mount | Docker socket access for API calls  |
| `/var/lib/docker/volumes` | Bind mount | Volume browsing via Portainer CE UI |

---

## CI/CD Deployment

This project uses GitHub Actions for continuous deployment, running on a **self-hosted runner**. On every push to the `main` branch, the workflow at `.github/workflows/deploy.yml`:

1. Checks out the latest code on the self-hosted runner.
2. Validates the Compose configuration with `docker compose config`.
3. Deploys/redeploys the agent with `docker compose up -d --remove-orphans`.

> **Note:** No secrets or `.env` file are required for this workflow — the agent container has no configurable credentials.

> **Note:** The workflow does not currently perform a post-deploy health check; it finishes as soon as `docker compose up -d` completes. Run `docker compose ps` on the server afterward to confirm the container is `running`.

A self-hosted GitHub Actions runner must be configured on the target server for this workflow to run.

---

## License

This project is licensed under the [MIT License](LICENSE).  
Copyright © 2026 Andry Pebrianto
