# Harbor Local Setup

## First Time Setup

```sh
# 1. Copy the config template
cp make/harbor.yml.tmpl make/harbor.yml
```

Edit `make/harbor.yml` — set `hostname: localhost`. If you don't have real certs, comment out the `https:` block and use HTTP only.

```sh
# 2. Compile Go binaries (runs inside a golang Docker container)
make compile

# 3. Build Docker images for all Harbor services
make build

# 4. Generate docker-compose.yml and config files from harbor.yml
#    Runs a privileged Docker container that writes files as root/UID 10000
make prepare

# 5. Fix permissions — prepare writes config files as root/UID 10000,
#    but container processes run as UID 10000 with no group match, so
#    they can't read 640 files. This makes them world-readable.
chmod -R a+r make/common/config/
find make/common/config/ -type d -exec chmod a+x {} \;

# 6. Start
make start
```

Harbor is now at **https://localhost** (or http://localhost if you disabled https).
Login: `admin` / `Harbor12345`

---

## After Making Code Changes

Only rebuild what changed — no need to rerun everything.

```sh
# Changed Go code (core, jobservice, registryctl)?
make compile       # recompile binaries
make build         # rebuild Docker images with new binaries

# Changed harbor.yml config?
make prepare
chmod -R a+r make/common/config/
find make/common/config/ -type d -exec chmod a+x {} \;

# Always finish with:
docker rm -f harbor-log harbor-jobservice harbor-core harbor-db harbor-portal registry registryctl nginx redis 2>/dev/null
make start
```

The `docker rm` before `make start` is required — if any previous run left containers behind (even stopped ones), `docker compose up` will fail with a name conflict.

---

## Stop / Restart

```sh
# Stop and remove containers (prompts for confirmation)
make down

# Then start again (no need to recompile or reprepare)
make start
```

---

## Useful logs

```sh
docker logs harbor-core -f
docker logs harbor-jobservice -f
docker logs harbor-log -f
docker logs harbor-db -f
```
