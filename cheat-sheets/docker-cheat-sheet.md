# Docker cheat sheet

Field-tested Docker and Compose commands for running servers – searchable and copyable in one click. With warnings on commands that delete data.

## Images

- List local images

  ```bash
  docker images
  ```
- Pull an image with a pinned tag

  ```bash
  docker pull nginx:1.27
  ```
- Build an image from the Dockerfile in this folder

  ```bash
  docker build -t myapp:1.0 .
  ```
- Retag an image for a registry

  ```bash
  docker tag myapp:1.0 registry.example.com/myapp:1.0
  ```
- Delete an image (**careful**)

  ```bash
  docker rmi myapp:1.0
  ```

## Running & managing containers

- List running containers

  ```bash
  docker ps
  ```
- List all containers including stopped ones

  ```bash
  docker ps -a
  ```
- Start a detached container with a port mapping

  ```bash
  docker run -d --name web -p 8080:80 nginx
  ```
- Throwaway container with an interactive shell

  ```bash
  docker run --rm -it alpine sh
  ```
- Restart the container after reboot or crash

  ```bash
  docker run -d --restart unless-stopped nginx
  ```
- Stop a container gracefully (SIGTERM, then SIGKILL)

  ```bash
  docker stop web
  ```
- Force-stop and delete a container (**careful**)

  ```bash
  docker rm -f web
  ```

## Logs, exec & diagnostics

- Last 100 log lines, then follow live

  ```bash
  docker logs -f --tail 100 web
  ```
- Open a shell in a running container

  ```bash
  docker exec -it web sh
  ```
- Run a single command in the container

  ```bash
  docker exec web nginx -t
  ```
- CPU and RAM usage of all containers (snapshot)

  ```bash
  docker stats --no-stream
  ```

## Volumes & networks

- Mount a named volume into a container

  ```bash
  docker run -v data:/var/lib/mysql mariadb
  ```
- Back up a volume to a tar archive

  ```bash
  docker run --rm -v data:/d -v $(pwd):/b alpine tar czf /b/data.tgz -C /d .
  ```
- Delete a volume – the data is gone (**careful**)

  ```bash
  docker volume rm data
  ```
- List networks

  ```bash
  docker network ls
  ```
- Create a user-defined bridge (DNS by container name)

  ```bash
  docker network create appnet
  ```

## Docker Compose

- Start the stack in the background

  ```bash
  docker compose up -d
  ```
- Rebuild images and start the stack

  ```bash
  docker compose up -d --build
  ```
- Follow the logs of one service live

  ```bash
  docker compose logs -f web
  ```
- Pull new images, recreate changed services

  ```bash
  docker compose pull && docker compose up -d
  ```
- Shell inside a service container

  ```bash
  docker compose exec web sh
  ```
- Stop the stack and remove containers and network

  ```bash
  docker compose down
  ```
- Like down, but also deletes the volumes (**careful**)

  ```bash
  docker compose down -v
  ```

## Cleaning up with prune

- Disk usage by images, containers, volumes

  ```bash
  docker system df
  ```
- Delete untagged (dangling) images (**careful**)

  ```bash
  docker image prune
  ```
- Delete all unused volumes – data loss possible (**careful**)

  ```bash
  docker volume prune
  ```
- Delete stopped containers, unused networks and dangling images (**careful**)

  ```bash
  docker system prune
  ```
- Radical: everything unused incl. images and volumes (**careful**)

  ```bash
  docker system prune -a --volumes
  ```

## Resource limits

- Limit memory to 512 MB

  ```bash
  docker run -d --memory 512m nginx
  ```
- Limit to 1.5 CPU cores

  ```bash
  docker run -d --cpus 1.5 nginx
  ```
- Change the limits of a running container

  ```bash
  docker update --memory 1g --memory-swap 1g web
  ```
- Set log rotation for a container

  ```bash
  docker run -d --log-opt max-size=10m --log-opt max-file=3 nginx
  ```
- Find OOM kills from the last hour

  ```bash
  docker events --since 1h --filter event=oom
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/docker-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/docker-cheat-sheet/)**
