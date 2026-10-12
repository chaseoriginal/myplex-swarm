# myplex-swarm Runbook

## Health Check

```bash
ssh ladmin@ore-dock-110-1 "docker service ps myplex-swarm_plex --no-trunc"
```

## Restart

```bash
ssh ladmin@ore-dock-110-1 \
  "docker service update --force myplex-swarm_plex"
```

## Logs

```bash
ssh ladmin@ore-dock-110-1 \
  "docker service logs myplex-swarm_plex --tail 50"
```

## Stop / Start

```bash
# Stop
ssh ladmin@ore-dock-110-1 "docker stack rm myplex-swarm"

# Start
ssh ladmin@ore-dock-110-1 \
  "cd /data/compose/myplex-swarm && docker stack deploy -c docker-compose.yml myplex-swarm"
```

## Recover from Crash Loop

1. Check logs: `docker service logs myplex-swarm_plex --tail 100`
2. Check volume mounts: `docker inspect myplex-swarm_plex`
3. If config corrupt: stop service, rename config dir, let Plex recreate from scratch, re-configure libraries

## Disaster Recovery

Named volume data is at:
- `/var/lib/docker/volumes/myplex_swarm_config/`
- `/var/lib/docker/volumes/myplex_swarm_transcode/`

These survive `docker stack rm`. Full disaster = reinstall from scratch with no named volume data.
