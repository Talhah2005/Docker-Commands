# Docker Commands

Quick reference for building, running, and debugging the Web Engineering Project with Docker Compose.

Docker Hub user: `talhah2005`
Images: `talhah2005/webeng-backend` and `talhah2005/webeng-frontend`

---

## 1. Build

```bash
docker compose build                 # build all images defined in docker-compose.yml
docker compose build backend         # build only one service
docker compose build --no-cache      # rebuild from scratch, ignoring cached layers
```

## 2. Start

```bash
docker compose up -d                 # start all containers in the background
docker compose up -d --build         # rebuild images, then start (use after code changes)
```

## 3. Check status

```bash
docker compose ps                    # list containers and their state (running / healthy)
docker network ls                    # list networks (look for frontend-net and backend-net)
docker volume ls                     # list volumes (look for mongo-data)
docker images                        # list images on this machine
```

## 4. Logs

```bash
docker compose logs backend          # print backend logs
docker compose logs mongo            # print mongo logs
docker compose logs frontend         # print frontend logs
docker compose logs -f backend       # follow live logs (Ctrl+C to stop following)
```

## 5. Stop

```bash
docker compose stop                  # stop containers, keep them for later restart
docker compose down                  # stop and remove containers and networks, keep data
docker compose down -v               # same as above AND delete volumes (database data is lost!)
```

## 6. Debugging

```bash
docker compose exec backend sh                      # open a shell inside the running backend container
docker compose restart backend                      # restart one service
docker network inspect webengineeringproject_backend-net   # see which containers are on a network
docker compose exec backend ping mongo              # should work (same network)
docker compose exec frontend ping mongo             # should fail (different network)
```

## 7. Push images to Docker Hub (on the laptop where the app is built)

```bash
docker login                                        # log in as talhah2005

docker compose build                                # build images (local names: webengineeringproject-backend / -frontend)

# Tag the local images with your Docker Hub name. Change 1.0 to the new version each release.
docker tag webengineeringproject-backend:latest talhah2005/webeng-backend:1.0
docker tag webengineeringproject-frontend:latest talhah2005/webeng-frontend:1.0

# Upload
docker push talhah2005/webeng-backend:1.0
docker push talhah2005/webeng-frontend:1.0
```

Check the result at https://hub.docker.com/u/talhah2005

### Pushing a new version (example: 2.0)

```bash
docker compose build
docker tag webengineeringproject-backend:latest talhah2005/webeng-backend:2.0
docker tag webengineeringproject-frontend:latest talhah2005/webeng-frontend:2.0
docker push talhah2005/webeng-backend:2.0
docker push talhah2005/webeng-frontend:2.0
```

Use a new tag for every release. Pushing the same tag again overwrites the old one.

### Optional: also push `latest`

```bash
docker tag webengineeringproject-backend:latest talhah2005/webeng-backend:latest
docker push talhah2005/webeng-backend:latest
```

## 8. Pull images from Docker Hub (on another laptop or the VPS)

```bash
docker pull talhah2005/webeng-backend:1.0           # download a specific version
docker pull talhah2005/webeng-frontend:1.0
docker images                                       # confirm they were downloaded
```

On the other machine you only need `docker-compose.yml` and `.env`, not the source code.
In `docker-compose.yml`, use `image:` instead of `build:`:

```yaml
backend:
  image: talhah2005/webeng-backend:1.0
frontend:
  image: talhah2005/webeng-frontend:1.0
```

```bash
docker compose pull                  # download the images listed in the compose file
docker compose up -d                 # start the containers
docker compose ps                    # confirm everything is running
```

Then open http://localhost in the browser.

### Updating to a new version on the other machine

```bash
# 1. change the tag in docker-compose.yml, e.g. :1.0 -> :2.0
docker compose pull
docker compose up -d                 # only containers with a new image are recreated
```

### Rolling back

```bash
# change the tag in docker-compose.yml back to :1.0, then
docker compose pull
docker compose up -d
```

## 9. Build a single image manually (without Compose)

```bash
docker build -t talhah2005/webeng-backend:1.0 ./Backend
docker build -t talhah2005/webeng-frontend:1.0 ./Frontend/vite-project
```

## 10. Docker Hub tips

```bash
docker logout                                       # log out of Docker Hub
docker rmi talhah2005/webeng-backend:1.0            # delete a local image
```

- Use an **access token** instead of your password: Docker Hub > Account Settings > Personal access tokens. Then run `docker login -u talhah2005` and paste the token as the password.
- Public repos are visible to everyone, so never put secrets or `.env` files inside an image.
- Always use version tags like `:1.0`, `:2.0` in production, so you can roll back by changing the tag.
- If the other laptop is an ARM machine (Mac M1/M2), build for both platforms:
  `docker buildx build --platform linux/amd64,linux/arm64 -t talhah2005/webeng-backend:1.0 --push ./Backend`

## 11. Cleanup

```bash
docker image prune                   # remove unused (dangling) images
docker system prune                  # remove stopped containers, unused networks, dangling images
docker system df                     # see how much disk space Docker is using
```

---

## Typical workflow

```bash
docker compose build                 # 1. build images
docker compose up -d                 # 2. start everything
docker compose ps                    # 3. confirm mongo is healthy, backend and frontend are running
docker compose logs -f backend       # 4. watch for errors
docker compose down                  # 5. stop when done
```
