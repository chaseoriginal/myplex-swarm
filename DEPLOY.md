# Deploy myplex-swarm

## Prerequisites

- Access to openclaw (ore-claw1)
- SSH access to ore-dock-110-1 as ladmin

## One-Time Setup

### 1. Sync repo to swarm leader

```bash
rsync -avz --delete ~/myplex-swarm/ ladmin@ore-dock-110-1:/data/compose/myplex-swarm/
ssh ladmin@ore-dock-110-1 "sudo chown -R ladmin:ladmin /data/compose"
```

### 2. Verify the .env

```bash
ssh ladmin@ore-dock-110-1 "cat /data/compose/myplex-swarm/.env"
```

### 3. Create the external network

```bash
ssh ladmin@ore-dock-110-1 \
  "docker network create --driver overlay --attachable myplex-swarm_public 2>/dev/null || echo network exists"
```

### 4. Verify placement label

```bash
ssh ladmin@ore-dock-110-1 "docker node ls"
# Ensure ore-dock-110-1 has label: deployment.target.myplex=true
# If not:
ssh ladmin@ore-dock-110-1 \
  "docker node update --label-add deployment.target.myplex=true ore-dock-110-1"
```

### 5. Deploy

```bash
ssh ladmin@ore-dock-110-1 \
  "cd /data/compose/myplex-swarm && docker stack deploy -c docker-compose.yml myplex-swarm"
```

## Redeploy (after changes)

```bash
# 1. Sync
rsync -avz --delete ~/myplex-swarm/ ladmin@ore-dock-110-1:/data/compose/myplex-swarm/

# 2. Redeploy
ssh ladmin@ore-dock-110-1 \
  "cd /data/compose/myplex-swarm && docker stack deploy -c docker-compose.yml myplex-swarm"
```

## Verify

```bash
ssh ladmin@ore-dock-110-1 "docker service ls | grep myplex"
ssh ladmin@ore-dock-110-1 "docker service ps myplex-swarm_plex"
ssh ladmin@ore-dock-110-1 \
  "curl -s http://localhost:32400/identity | python3 -c \
    \"import sys,json; d=json.load(sys.stdin); print(d.get('mediaContainer',{}).get('friendlyName','?'))\""
```
