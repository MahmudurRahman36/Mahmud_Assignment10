# Assignment 10 — Dockerize & Deploy Express.js with Docker Compose + Nginx

This guide explains, step by step, how to containerize this Express.js app, push the
image to DockerHub, and run it alongside Nginx using Docker Compose. Follow the steps
in order and you can reproduce the whole setup yourself.

> Replace the DockerHub username **`mrkolince`** with your own everywhere it appears.

---

## Prerequisites

- **Docker Desktop** installed and running on Windows (provides Docker Engine + Docker Compose v2).
- A **DockerHub account**.
- **Git** installed.
- A terminal (this guide uses **Windows PowerShell**).

---

## How the pieces fit together

Docker Compose builds the Express app from the **Dockerfile** and pulls a stock
**nginx:latest** image. The app container maps host port **8080** to container port
**5000** (where Express listens). `depends_on` makes Compose start **Nginx first**,
then the Express app.

---

## Step 1 — Clone the repository

```bash
git clone https://github.com/roy35-909/Module-3-deployment
cd Module-3-deployment
ls
```

## Step 2 — The `Dockerfile`

Already included in this repo:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 5000
ENV PORT=5000
CMD ["npm", "start"]
```

- `node:18-alpine` — small Node.js base image.
- `npm install --production` — installs only runtime dependencies.
- `EXPOSE 5000` / `ENV PORT=5000` — the app listens on port 5000 inside the container.

## Step 3 — The `docker-compose.yml`

Already included in this repo:

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx-server
    restart: always

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: express-app
    ports:
      - "8080:5000"
    environment:
      - PORT=5000
    depends_on:
      - nginx
    restart: always
```

- **nginx** — runs a plain `nginx:latest` image.
- **app** — built from our `Dockerfile`.
- `ports: "8080:5000"` — exposes the app on host port **8080**.
- `depends_on: nginx` — the Express app starts **after** Nginx.

## Step 4 — Build the image

```bash
docker build -t mrkolince/mahmud_assignment10:latest .
docker images mrkolince/mahmud_assignment10
```

## Step 5 — Push to DockerHub

```bash
docker login
docker push mrkolince/mahmud_assignment10:latest
```

The image is published at:
**https://hub.docker.com/r/mrkolince/mahmud_assignment10**

## Step 6 — Start everything with Docker Compose

```bash
docker compose up -d
```

Compose pulls `nginx:latest`, builds the app, creates a network, and starts both
containers (Nginx first, then the Express app).

## Step 7 — Verify the running containers

```bash
docker compose ps
docker ps
```

You should see `express-app` with `0.0.0.0:8080->5000/tcp` and `nginx-server` on `80/tcp`.

## Step 8 — Verify in the browser

Open: **http://localhost:8080**

The Express app responds — confirming it is reachable on port 8080.

## Step 9 — Confirm on DockerHub

Open your repository and confirm the `latest` tag is present:

```bash
docker pull mrkolince/mahmud_assignment10:latest
```

---

## Useful commands

```bash
docker compose logs -f      # stream logs from both containers
docker compose down         # stop and remove the containers + network
docker compose up -d --build  # rebuild the app image and restart
```

---

## Submission summary

| Item | Value |
|------|-------|
| My Git repository | https://github.com/MahmudurRahman36/Mahmud_Assignment10 |
| Source repository | https://github.com/roy35-909/Module-3-deployment |
| DockerHub image | https://hub.docker.com/r/mrkolince/mahmud_assignment10 |
| Pull command | `docker pull mrkolince/mahmud_assignment10:latest` |
| Files added | `Dockerfile`, `docker-compose.yml` |
| App URL | http://localhost:8080 |
| Exposed port | 8080 (host) → 5000 (container) |
