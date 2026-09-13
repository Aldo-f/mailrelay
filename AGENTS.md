# AGENTS.md — 06-apps-mailrelay

Internal SMTP relay for the Pi home-lab. No public web UI — other containers join
the `mailrelay` Docker network and send mail via `host=mailrelay, port=25`.

## Repo shape

```
06-apps-mailrelay/
├── docker-compose.yml   ← the service definition (crazymax/msmtpd)
├── .env.example         ← template; copy to .env and fill credentials
├── README.md            ← usage docs (how other containers use it)
└── .gitignore           ← excludes .env
```

## Commands

```bash
# Start the relay
cd ~/dev/06-apps-mailrelay && docker compose up -d

# Check status
docker ps --filter name=mailrelay
docker logs mailrelay -f    # follow logs
docker logs mailrelay | grep smtpstatus  # see delivery results

# Restart after .env change
docker compose up -d --no-deps mailrelay
```

## How other containers use it

Add to their docker-compose.yml:

```yaml
services:
  myapp:
    # ...
    networks:
      - mailrelay

networks:
  mailrelay:
    external: true
```

Then point any SMTP library at `host=mailrelay, port=25` (no auth needed).

For WordPress-style sendmail compatibility, install msmtp and copy an msmtprc:
```dockerfile
RUN apt-get update && apt-get install -y msmtp
COPY msmtprc /etc/msmtprc
RUN ln -sf /usr/bin/msmtp /usr/sbin/sendmail
```

With `/etc/msmtprc`:
```
account default
host mailrelay
port 25
tls off
from your-app@localhost
logfile -
```

## Credentials

Gmail SMTP credentials live in `.env` — **never commit that file**.
To rotate password or change provider: edit `.env`, then `docker compose up -d --no-deps mailrelay`.

## Open questions

- Should this be added to `~/dev/AGENTS.md` submodule map? (currently missing)
- Should Traefik route be added for web-based log viewer? (not needed yet)
