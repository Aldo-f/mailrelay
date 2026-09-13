# SMTP Relay

Internal SMTP relay using `crazymax/msmtpd`. Forwards all mail to Gmail SMTP — no host port is published, so the relay is only reachable from other containers on the same Docker network.

## Upstream

| Setting | Value |
|---------|-------|
| Host | `smtp.gmail.com` |
| Port | `587` |
| TLS | on (STARTTLS) |
| Auth | on |

Credentials are in `.env` — **never commit that file**.

## How other containers send mail

1. Attach the container to the `mailrelay` network.
2. Set SMTP host to `mailrelay` and port `25`.
3. No authentication needed.

### Docker Compose service snippet

```yaml
services:
  myapp:
    image: myapp:latest
    networks:
      - mailrelay   # join the relay network

networks:
  mailrelay:
    external: true  # refers to the network created by mailrelay/compose
```

### Plain `docker run`

```bash
docker run --network mailrelay myapp
```

Inside the container, point your mail library at:

```
host = mailrelay
port = 25
```

## Testing

### Start the relay

```bash
cd ~/dev/06-apps-mailrelay
docker compose up -d
```

### Send a test email from a throwaway container

```bash
docker run --rm --network mailrelay python:3.12-slim \
  python -c "
import smtplib
from email.message import Message
msg = Message()
msg['From'] = 'mailrelay@localhost'
msg['To'] = 'aldo.fieuw@gmail.com'
msg['Subject'] = 'SMTP Relay Test'
msg.set_payload('If you received this, the relay is working.')
s = smtplib.SMTP('mailrelay', 25)
s.send_message(msg)
s.quit()
print('sent')
"
```

### View relay logs

```bash
docker logs mailrelay -f
```

Look for lines containing `accepted` or `sent` to confirm the upstream accepted the message.

## Rotate password / change provider

1. Edit `.env` with the new credentials.
2. Restart the container (no need to rebuild):

```bash
docker compose up -d --no-deps mailrelay
```

3. Check logs: `docker logs mailrelay | tail -20`

## Network

The `mailrelay` network is created with `name: mailrelay` in compose, making it externally referenceable. Verify with:

```bash
docker network ls | grep mailrelay
```
