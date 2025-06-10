# macjuu-public
This repository contains a generalized version* of the code used to build and maintain https://macjuu.com; a 10+ year-old Macbook Air powered by Ubuntu & Docker. Given its age, it can die any moment, so I built it from code.

*: mainly, all `.env` files were scrubbed from personal info and tokens, available as `.env.example` to re-build your own version from.

Below info is far from complete but should give a good starting point in case you want to build your own selfhosted stacks.

Source repo: [github.com/niels-emmer/macjuu-public](https://github.com/niels-emmer/macjuu-public)

### Service stack

- Reverse proxy/SSL: [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager), [LetsEncrypt](https://letsencrypt.org/)
- Auth: [Authentik](https://github.com/goauthentik/authentik)
- SecOps: [Fail2ban](https://github.com/fail2ban/fail2ban), [Grafana/Loki](https://github.com/grafana/grafana)
- Orchestration: [Komodo](https://github.com/moghtech/komodo)
- Backup: [BackRest](https://github.com/garethgeorge/backrest)
- Monitoring: [Uptime-Kuma](https://github.com/louislam/uptime-kuma), [ntfy](https://github.com/binwiederhier/ntfy)

### Repo structure

| Folder | Contents |
| ------ | ------- |
| base/nginx | docker stack with Nginx Proxy Manager and Fail2ban |
| base/komodo | docker stack for komodo (to deploy other stacks) |
| stacks | all docker stacks in a folder per stack |
| scripts | some useful shell scripts |

## principles
1. All public facing services should use HTTPS/TLS
2. Access to private services and/or storing private information utilize MFA (thanks, Authentik)
3. All non-transient data is stored on SSD in `$DOCKER_DATA/$stack-name` 
4. Large (cool) data can be stored on HDD in `$DOCKER_EXTDATA/$stack-name`
5. Stack directories should only contain `compose.yaml` and .`env` files, no persistent data.

## prerequisites
- Recent Ubuntu Server (or equivalent) install with ssh or terminal access.
- Docker (+ compose plugin) ([on linux](https://docs.docker.com/engine/install/ubuntu/))
- GIT

## 1. Clone this repo
```
git clone https://github.com/niels-emmer/macjuu-public.git
cd macjuu-public
```

## 2. Restore persistent data

All apps need to have persistent data available in 
- `$DOCKER_DATA`, optimally an SSD, for configuration files, (small) databases etc, and
- `$DOCKER_EXTDATA`, which can be an HDD, for large files, media etc..

Hot data is synced to `/mnt/backup/dockerdata` on a daily basis and can be restored with:
```
sudo rsync -a /mnt/backup/dockerdata /home/userdir/dockerdata
```

## 3. Setup Base
The first two stacks are handled outside of Komodo. Need a base, after all. The Nginx stack deploys Nginx-Proxy-Manager (Nginx Reverse Proxy with a web UI) and fail2ban to deter scriptkiddies. The Komodo stack deploys Komodo, a Docker orchestrator that will deploy all other stacks.

![Komodo step 1](https://github.com/niels-emmer/macjuu-public/blob/main/share/komodo1.png)

### Set base env
```
export DOCKER_DATA=/home/zaph0d/dockerdata
export DOCKER_EXTDATA=/mnt/dockerdata
```

### Install nginx-proxy-manager, fail2ban and goaccess

1. Edit, if needed, `base/niginx/.env` to reflect storage locations et all, and
2. Start the stack

```
cd base/nginx
cp .env.example .env
nano .env
docker compose -p nginx up -d
cd ../..
```

### Install komodo

1. Edit, if needed, `base/komodo/.env` to reflect storage locations et all, and
2. Start the stack

```
cd base/komodo
cp .env.example .env
nano .env
docker compose -p komodo up -d
cd ../..
```

## 4. Add stacks from Komodo

Komodo can be found at http://[dockerhost-ip]:9120. It is setup to sync its configuration to `main.toml` in the root of this repository. This configuration includes stack definitions for each workload, sourced from the `stacks` folder in this same repo.

![Komodo step 2](https://github.com/niels-emmer-public/macjuu/blob/main/share/komodo2.png)

1. If needed, import full configuration from [main.toml](https://github.com/niels-emmer/macjuu-public/blob/main/main.toml). This will setup server and all stack definitions, install the notifier and sync.
2. Then deploy stacks as needed. Each stack will be pulled from `github.com/niels-emmer/macjuu-public` and run the compose file in `stacks/$stackname`, which is also set as run-directory and will pick up `.env` and config files when deployed by Komodo.



## snippets

- [Custom NGINX code to provide SSO to any app, using Authentik outpost](https://github.com/niels-emmer/macjuu-public/blob/main/share/snippets/npm-custom-code-outpost) 
