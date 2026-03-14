# Docker & Docker Compose Notes

These notes explain **Docker / Docker Compose** and provide *minimal working config* you can use (main focus on spring applications).

---

## What is Docker?

**Docker** lets you package an application + its runtime + dependencies into a **container image**.

Why it’s useful:
- Same behavior across machines (dev/stage/prod)
- No need to install different software on the host
- Easy deployments (run the same image anywhere Docker runs)

Key terms:
- **Image**: the blueprint/snapshot of working app (built from a `Dockerfile`)
- **Container**: a running instance of an image (used to run app instead of stratch build)
- **Dockerfile**: instructions to build an image of app

---

## What is Docker Compose?

**Docker Compose** runs **multiple containers together** (for full/multiple stack app) using a single `docker-compose.yml`.

Why it’s useful:
- One command to start app + database + cache, etc.
- Built-in networking (services can talk by service name)
- Central place for ports, environment variables, volumes

---

# Minimal Docker Example (single service)

This example runs a Spring Boot JAR inside a container.

## 1) Minimal `Dockerfile`

```dockerfile
# Use an image of JDK to run your Spring Boot jar
FROM eclipse-temurin:21-jdk

#sets the current working directory (like cd in terminal)
WORKDIR /app

#
COPY app.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

## 2) Build the image

```bash
docker build -t my-spring-app .
```

## 3) Run the container

```bash
docker run --rm -p 8080:8080 --name my-spring-app my-spring-app
```

Open:
- http://localhost:8080

---

# Minimal Docker Compose Example (app + database)

This example runs:
- a MySQL container
- an spring/app container

## 1) Minimal `docker-compose.yml`

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: appdb
    ports:
      - "3306:3306"

  app:

    # builds app docker file using "Dockerfile"
    build: .
    ports:
      - "8080:8080"
    environment:
      # Example environment variables you might use in Spring:
      SPRING_DATASOURCE_URL: jdbc:mysql://db:3306/appdb
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: example
    depends_on:
      - db
```

Important concept: inside Compose, the hostname `db` works because Compose creates a network and registers service names in DNS.

## 2) Start the stack

```bash
docker compose up --build
```

Stop it:

```bash
docker compose down
```

---

## Minimal command cheat-sheet

Build image:
```bash
docker build -t my-image .
```

Run container:
```bash
docker run --rm -p 8080:8080 my-image
```

Compose up:
```bash
docker compose up --build
```

```bash
docker compose up --build -d #runs in background
```

Compose down:
```bash
docker compose down
```

Logs:
```bash
docker compose logs -f
```

---

## Notes / Good practices (short)

- Don’t hardcode passwords in real projects—use a `.env` file for prod/dev/docker .
- Use volumes for database persistence (otherwise data resets when container is removed).
- Prefer multi-stage builds for Java apps (build with Maven/Gradle in one stage, run on a smaller base image).