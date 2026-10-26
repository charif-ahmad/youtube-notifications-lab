# SSH Access and Server Baseline

**Recorded:** 2026-10-26  
**Status:** Initial access and host checks completed

This note records the repeatable setup path. Machine-specific values are intentionally replaced with placeholders; the local connection note is kept outside Git.

## Laptop to home server

An Ed25519 key was created on the development laptop. Its public key was installed on the home server with `ssh-copy-id`, then key-based login was confirmed.

```bash
ssh-keygen -t ed25519 -C "dev-laptop-to-home-server"
ssh-copy-id -i ~/.ssh/id_ed25519.pub <server-user>@<server-lan-ip>
ssh <server-user>@<server-lan-ip>
```

The laptop's private key stays on the laptop. Do not copy it into the repository or onto the server.

## Home server to GitHub

A separate Ed25519 key was created on the server for this repository. Its public key was added under the repository's **Settings → Deploy keys** with write access disabled. Authentication was confirmed with:

```bash
ssh -T -i ~/.ssh/youtube_notifications_deploy \
  -o IdentitiesOnly=yes git@github.com
```

GitHub confirmed authentication for this repository. The server-side private key remains under the server user's `.ssh` directory and is not part of this repository.

The repository was cloned on the server into `~/youtube-notifications-lab`; the checked-out branch was `main` tracking `origin/main`.

## Initial server inventory

The following values were observed on 2026-10-26. Memory availability and container state can change, so recheck before load tests.

| Item | Observed value |
| --- | --- |
| Operating system | Linux Mint 22.3 |
| Kernel | Linux 6.17.0-19-generic |
| Architecture | x86-64 |
| Processor count | 4 logical CPUs |
| Memory | 3.8 GiB total; 2.6 GiB available after stopping the existing application containers |
| Root filesystem | 457 GB total; 388 GB available |
| Git | 2.43.0 |
| Docker Engine | 29.1.3 |
| Docker Compose | 2.40.3+ds1-0ubuntu1~24.04.1 |
| Docker service | Active |
| Docker CLI permissions | `sudo` is currently required |

Three pre-existing OneCloudSync containers (frontend, backend, PostgreSQL) were stopped to reduce background workload before lab experiments. They were stopped, not removed; their container data was not intentionally deleted.

## Current deployment status

GitHub access from the server works and the repository clone is present. The repository currently contains project notes and plans; the application has not yet been deployed to the server.
