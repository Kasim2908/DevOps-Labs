# Docker Compose Advanced Commands

## Build Images

```bash
docker compose build
```

---

## Build Without Cache

```bash
docker compose build --no-cache
```

---

## Start Containers

```bash
docker compose up
```

---

## Start in Detached Mode

```bash
docker compose up -d
```

---

## Stop Containers

```bash
docker compose down
```

---

## Stop and Remove Volumes

```bash
docker compose down -v
```

---

## Force Recreate Containers

```bash
docker compose up -d --force-recreate
```

---

## Show Running Containers

```bash
docker compose ps
```

---

## View Logs

```bash
docker compose logs
```

---

## Follow Logs

```bash
docker compose logs -f
```

---

## Execute Command Inside Container

```bash
docker compose exec web bash
```

---

## View Docker Networks

```bash
docker network ls
```

---

## View Docker Volumes

```bash
docker volume ls
```

---

## Validate Compose File

```bash
docker compose config
```
