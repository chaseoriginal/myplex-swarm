# myplex-swarm

Plex Media Server on Swarm 110-1 (ore-dock-110-1).

Stack name: `myplex-swarm` | Service: `plex` | Image: `lscr.io/linuxserver/plex:latest`

## Architecture

- **Host:** ore-dock-110-1 (Swarm 110-1 leader, 10.77.110.10)
- **Node label:** `deployment.target.myplex=true`
- **Advertise URL:** https://myplex.echoinggreen.net:32400/
- **Access:** https://myplex.echoinggreen.net (reverse-proxied)
- **Engine:** Docker 29.9.0

## Volumes

| Volume | Type | Mount | Purpose |
|--------|------|-------|---------|
| `myplex_swarm_config` | Named (pre-existing) | `/config` | Plex config, metadata, plugin data |
| `myplex_swarm_transcode` | Named (pre-existing) | `/transcode` | Transcoding temp files |
| `/media/Multimedia` | Bind mount | `/media` | Main media library |
| `/media/Multimedia/Music` | Bind mount | `/music` | Music library |

Named volumes are declared in compose and will reuse the existing data at:
- `/var/lib/docker/volumes/myplex_swarm_config/`
- `/var/lib/docker/volumes/myplex_swarm_transcode/`

## Named Volume Conversion

The original config/transcode paths were bind mounts to `/var/lib/docker/volumes/myplex_swarm/{config,transcode}`.
These are now properly declared as named volumes in compose — data is preserved on redeploy.

## Environment Variables

See `.env.example`. Key variables:
- `TZ=America/Chicago`
- `IP=myplex.echoinggreen.net`
- `CNAME=myplex_swarm`
- `PUID=1000`, `PGID=1000`
- `PLEX_CLAIM=` — not required (server is already claimed)

## Deployment Model

```
openclaw (ladmin) ──rsync──> ore-dock-110-1 (swarm leader)
                                     │
                                     └── docker stack deploy
```

Changes flow: edit on openclaw → commit → rsync to swarm leader → `docker stack deploy`.

## Portainer Integration

This stack is NOT managed by Portainer (deployed directly via `docker stack`).
After deploy, update Portainer via API to associate this compose with a Portainer stack entry.

## Upgrading

```bash
# Pull latest image
docker pull lscr.io/linuxserver/plex:latest

# Redeploy
cd /data/compose/myplex-swarm
docker stack deploy -c docker-compose.yml myplex-swarm
```
