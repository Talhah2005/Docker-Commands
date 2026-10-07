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

## 8. Push to Docker Hub

```bash
docker login                                              # log in to Docker Hub
docker tag zakat-backend yourusername/zakat-backend:1.0   # name the image for your account
docker push yourusername/zakat-backend:1.0                # upload it
```

## 9. Deploy on a VPS (after pushing images)

```bash
docker compose pull                  # download the latest images
docker compose up -d                 # start or update containers
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
