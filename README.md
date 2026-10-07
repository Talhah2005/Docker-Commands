# Docker Commands

Quick reference for building, running, and debugging a multi-container app with Docker Compose.

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
docker compose exec backend sh       # open a shell inside the running backend container
docker compose restart backend       # restart one service
docker network inspect <project>_backend-net   # see which containers are on a network
docker compose exec backend ping mongo         # should work (same network)
docker compose exec frontend ping mongo        # should fail (different network)
```

## 7. Build a single image manually

```bash
docker build -t zakat-backend ./Backend                  # build backend image
docker build -t zakat-frontend ./Frontend/vite-project   # build frontend image
```
## 8. Push images to Docker Hub

Do this on your laptop, after the images are built.

```bash
docker login                                                # log in (use your Docker Hub username + password or access token)

docker build -t zakat-backend ./Backend                     # build the image locally (skip if already built)
docker build -t zakat-frontend ./Frontend/vite-project

docker tag zakat-backend yourusername/zakat-backend:1.0     # rename it as <username>/<repo>:<version>
docker tag zakat-frontend yourusername/zakat-frontend:1.0

docker push yourusername/zakat-backend:1.0                  # upload to Docker Hub
docker push yourusername/zakat-frontend:1.0
```

Optionally also tag and push `latest`, so a plain pull gets the newest version:

```bash
docker tag zakat-backend yourusername/zakat-backend:latest
docker push yourusername/zakat-backend:latest
```

Check the result at `https://hub.docker.com/repositories/yourusername`.

### Shortcut with Compose

Add an `image:` name next to `build:` in `docker-compose.yml`:

```yaml
backend:
  build: ./Backend
  image: yourusername/zakat-backend:1.0

frontend:
  build: ./Frontend/vite-project
  image: yourusername/zakat-frontend:1.0
```

Then one command builds, tags, and pushes everything:

```bash
docker compose build                 # builds and tags with the image: names
docker compose push                  # pushes all services that have an image: name
```

## 9. Pull images from Docker Hub

Do this on the VPS or any other machine.

```bash
docker login                                                # only needed for private repositories
docker pull yourusername/zakat-backend:1.0                  # download a specific version
docker pull yourusername/zakat-frontend:1.0
docker pull yourusername/zakat-backend                      # no tag means :latest
docker images                                               # list downloaded images
```

Run a pulled image directly (without Compose):

```bash
docker run -d -p 5000:5000 --name backend yourusername/zakat-backend:1.0
```

### Deploy with Compose on the VPS

On the VPS you only need `docker-compose.yml` and `.env`, not the source code. Use `image:` and remove the `build:` lines, so the file has:

```yaml
backend:
  image: yourusername/zakat-backend:1.0
frontend:
  image: yourusername/zakat-frontend:1.0
```

```bash
docker compose pull                  # download the latest versions of all images
docker compose up -d                 # start or update the containers
docker compose ps                    # confirm everything is running
```

### Updating after a code change

```bash
# On your laptop
docker compose build
docker compose push

# On the VPS
docker compose pull
docker compose up -d                 # only containers with a new image are recreated
```

## 10. Docker Hub tips

```bash
docker logout                        # log out of Docker Hub
docker search nginx                  # search public images from the terminal
docker rmi yourusername/zakat-backend:1.0   # delete a local image
```

- Use an **access token** instead of your password: Docker Hub > Account Settings > Personal access tokens. Then run `docker login -u yourusername` and paste the token as the password.
- Free accounts get one private repository. Public repos are visible to everyone, so never put secrets or `.env` files in an image.
- Always use version tags like `:1.0`, `:1.1` in production, so you can roll back by changing the tag.
```

## 10. Cleanup

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
