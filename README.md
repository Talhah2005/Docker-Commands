# Docker Commands (Reusable Cheat Sheet)

Quick reference for building, running, pushing, and debugging any multi-container app with Docker Compose.

Docker Hub user: `talhah2005`

## Placeholders: what to replace

| Placeholder | Meaning | Example |
|---|---|---|
| `<service>` | A service name from `docker-compose.yml` | `backend`, `frontend`, `mongo` |
| `<project-folder>` | Name of the folder that holds `docker-compose.yml`, lowercase, no spaces. Compose uses it to name local images, networks, and volumes. | `myapp` |
| `<image-name>` | The repository name you choose on Docker Hub | `myapp-backend` |
| `<version>` | The version tag for a release | `1.0`, `2.0` |
| `<path-to-dockerfile-folder>` | Folder containing the Dockerfile | `./Backend` |
| `<network-name>` | A network name from `docker-compose.yml` | `backend-net` |

Local image names created by Compose follow the pattern `<project-folder>-<service>`. Run `docker images` to see the exact names.

---

## 1. Build

```bash
docker compose build                       # build all images defined in docker-compose.yml
docker compose build <service>             # build only one service
docker compose build --no-cache            # rebuild from scratch, ignoring cached layers
```

## 2. Start

```bash
docker compose up -d                       # start all containers in the background
docker compose up -d --build               # rebuild images, then start (use after code changes)
```

## 3. Check status

```bash
docker compose ps                          # list containers and their state (running / healthy)
docker network ls                          # list networks
docker volume ls                           # list volumes
docker images                              # list images on this machine
```

## 4. Logs

```bash
docker compose logs <service>              # print logs of one service
docker compose logs -f <service>           # follow live logs (Ctrl+C to stop following)
docker compose logs                        # logs of all services
```

## 5. Stop

```bash
docker compose stop                        # stop containers, keep them for later restart
docker compose down                        # stop and remove containers and networks, keep data
docker compose down -v                     # same as above AND delete volumes (data is lost!)
```

## 6. Debugging

```bash
docker compose exec <service> sh           # open a shell inside a running container
docker compose restart <service>           # restart one service
docker network inspect <project-folder>_<network-name>   # see which containers are on a network
docker compose exec <service> ping <other-service>       # test whether two services can reach each other
```

Edit: use real service names from your compose file.

## 7. Push images to Docker Hub

Do this on the machine where the app is built.

```bash
docker login                               # log in as talhah2005 (or use an access token)
docker compose build                       # build images

# Tag each local image: docker tag <local-image> talhah2005/<image-name>:<version>
docker tag <project-folder>-<service>:latest talhah2005/<image-name>:<version>

# Upload
docker push talhah2005/<image-name>:<version>
```

Edit: do the `tag` and `push` lines once per service that you want on Docker Hub (for example backend and frontend). Skip services that use public images, like `mongo`.

Check the result at https://hub.docker.com/u/talhah2005

### Pushing a new version

Change only `<version>` (for example 1.0 to 2.0) and repeat the same three steps: build, tag, push. Use a new tag for every release, because pushing an existing tag overwrites it.

### Optional: also push `latest`

```bash
docker tag <project-folder>-<service>:latest talhah2005/<image-name>:latest
docker push talhah2005/<image-name>:latest
```

## 8. Pull images from Docker Hub

Do this on another laptop or on a VPS. You only need `docker-compose.yml` and `.env`, not the source code.

```bash
docker login                               # only needed for private repositories
docker pull talhah2005/<image-name>:<version>   # download one image
docker images                              # confirm it was downloaded
```

In `docker-compose.yml`, use `image:` instead of `build:`:

```yaml
<service>:
  image: talhah2005/<image-name>:<version>
```

Edit: do this for each of your own services. Keep the service names exactly the same as in the original compose file, because other parts of the app (like Nginx config or connection strings) may refer to them by name.

```bash
docker compose pull                        # download all images listed in the compose file
docker compose up -d                       # start the containers
docker compose ps                          # confirm everything is running
```

### Updating to a new version

```bash
# 1. change the tag in docker-compose.yml, e.g. :1.0 -> :2.0
docker compose pull
docker compose up -d                       # only containers with a new image are recreated
```

### Rolling back

Change the tag in `docker-compose.yml` back to the old version, then run `docker compose pull` and `docker compose up -d`.

## 9. Build a single image manually (without Compose)

```bash
docker build -t talhah2005/<image-name>:<version> <path-to-dockerfile-folder>
```

Edit: the path is the folder that contains the Dockerfile, for example `./Backend`.

## 10. Docker Hub tips

```bash
docker logout                              # log out of Docker Hub
docker rmi talhah2005/<image-name>:<version>   # delete a local image
```

- Use an **access token** instead of your password: Docker Hub > Account Settings > Personal access tokens. Then run `docker login -u talhah2005` and paste the token as the password.
- Public repos are visible to everyone, so never put secrets or `.env` files inside an image. Check that `.env` is listed in `.dockerignore`.
- Always use version tags in production, so you can roll back by changing the tag.
- If the other machine is ARM (Mac M1/M2), build for both platforms:
  `docker buildx build --platform linux/amd64,linux/arm64 -t talhah2005/<image-name>:<version> --push <path-to-dockerfile-folder>`

## 11. Cleanup

```bash
docker image prune                         # remove unused (dangling) images
docker system prune                        # remove stopped containers, unused networks, dangling images
docker system df                           # see how much disk space Docker is using
```

---

## Typical workflow

```bash
docker compose build                       # 1. build images
docker compose up -d                       # 2. start everything
docker compose ps                          # 3. confirm services are running / healthy
docker compose logs -f <service>           # 4. watch for errors
docker compose down                        # 5. stop when done
```

## New project checklist

1. Each service has a Dockerfile and a `.dockerignore` (include `node_modules`, `.env`, `.git`).
2. Secrets go in `.env`, and `.env` is in `.gitignore`.
3. `docker-compose.yml` defines services, networks, volumes, and healthchecks.
4. Run `docker compose up -d --build` and test locally.
5. Tag, push, then pull and test on a second machine or the VPS.
