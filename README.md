# Home Server Setup

This repository contains the configuration, scripts, and Docker Compose setup for my personal home server. The stack is designed to provide media management, file sharing, monitoring, and torrenting in a simple, reproducible way.

---

## Stack Overview

The server currently runs **six containers**:

| Service        | Image                        | Purpose                                                                 |
|----------------|------------------------------|-------------------------------------------------------------------------|
| Jellyfin       | `jellyfin/jellyfin:latest`   | Media server for streaming movies, TV, and music                        |
| Netdata        | `netdata/netdata:latest`     | System and service monitoring                                           |
| Nextcloud      | `nextcloud:latest`           | File sharing and cloud storage                                           |
| Organize       | `organize:arm64`             | Automatic media organization                                            |
| PostgreSQL     | `postgres:14`                | Database for applications like Nextcloud                                |
| qBittorrent    | `linuxserver/qbittorrent`    | Torrent client for downloads                                            |

---

## Directory Structure

home-server/
├── docker-compose.yml # Main Docker Compose file
├── .gitignore # Ignore runtime data and logs
├── README.md # This file
├── configs/ # Versioned container configs
│ ├── organize/
│ │ └── config.yml # Organize rules
│ └── qbittorrent/
│ └── config/ # qBittorrent config files
├── scripts/ # Utility scripts / cron jobs
│ ├── organize-run.sh # Wrapper to run Organize and log
│ └── check-organize.sh # Check previous day's Organize run
└── data/ # Placeholder for persistent volumes
├── postgres/ # PostgreSQL volume
└── nextcloud/ # Nextcloud volume

markdown
Copy code

**Notes:**

- Configs in `configs/` are versioned; runtime data goes in `data/` and is ignored by Git.
- Scripts in `scripts/` handle cron job automation and logging.
- Logs and temporary files are excluded from the repo.

---

## Organize Container

The Organize container automatically moves media from the downloads folder to your media library.

- The container **does not run as a daemon**; instead, it is triggered by a cron job.
- Cron schedule: runs **daily at 8 PM**.
- Log file: `~/docker/organize/cron.log`
- Previous runs are validated by checking the log for a `"success"` message.

**Example Cron Entry:**

```cron
# Run Organize daily at 8 PM
0 20 * * * cd ~/docker && /usr/bin/docker compose run --rm organize >> ~/docker/organize/cron.log 2>&1```
PostgreSQL Notes
The PostgreSQL container uses a dedicated persistent volume (data/postgres).

Ensure proper file ownership: the directory must be owned by the postgres user to avoid initialization errors.

Restart policies in docker-compose.yml are set to ensure containers start cleanly without infinite restarts due to misconfiguration.

Logging and Maintenance
Organize logs are rotated to keep the last 7 days.

System monitoring is available via Netdata at port 19999.

Containers are managed through Docker Compose.

Setup Instructions
Clone the repo:

bash
Copy code
git clone <your-github-repo-url> home-server
cd home-server
Copy or adjust configs in configs/ as needed.

Start containers:

bash
Copy code
docker compose up -d
Check container health:

bash
Copy code
docker compose ps
Verify that logs for Organize, PostgreSQL, and other services show no errors.

Future Improvements
Optional: email notifications for failed Organize runs.

Optional: additional automated scripts for backups.

Further containerized apps may be added as needed (e.g., Nextcloud plugins, alternative media tools).
