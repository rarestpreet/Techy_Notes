# Deployment Notes (in reference to `Spring-Docker-template`)

These notes reflect what I learned by building a **Spring Boot backend + MySQL database** and running them reliably using **Docker** and **Docker Compose** (the same pattern used in `rarestpreet/Spring-Docker-template`).

---

## 1) What “deployment” means

**Deployment** is the process of making an application available outside your development machine so users/services can access it.

Typical architecture:

**Frontend → Backend API → Database**

Example:

**React app → Spring Boot API → MySQL**

In `Spring-Docker-template`, I mainly practiced deploying (running) the **backend + database** as a working system.

---

## 2) What I deployed in `Spring-Docker-template`

### Backend (Spring Boot)
- Runs as a server application (REST API).
- Listens on **8080** inside the container.

### Database (MySQL 8)
- Runs as a separate service/container.
- MySQL listens on **3306** inside its container.
- (Optional but common) MySQL can be mapped to a different host port (example: `3307:3306`) to avoid conflicts with local MySQL.

### Key takeaway
The backend should not depend on “local installs” (Java, Maven, MySQL). Docker/Compose becomes the **runtime contract** for running the app.

---

## 3) Why Docker matters (for deployment)

Docker helps you ship your backend as a **container image**, which makes it easier to deploy on platforms like Railway/Render/Fly/AWS because:

- the server doesn’t need custom setup steps (Java/Maven installs, etc.)
- the platform can follow your **Dockerfile** to build and run the app
- the same image can run in multiple places (local, staging, production)

**Mental model**
- `Dockerfile` = how to package and run **one service** (your backend).

---

## 4) Why Docker Compose matters (mostly for development)

Your backend isn’t “just Spring” — it needs MySQL.

Docker Compose gives you:
- one command to run **spring + mysql**
- a shared network where services can communicate using **service names**
- one file for ports + environment variables

**Mental model**
- `docker-compose.yml` = how to run the **whole system** (backend + database) together.

> In production, many teams use managed databases and platform tooling instead of `docker-compose.yml`, but the *concept* remains the same: multiple services working together.

---

## 5) Compose networking concept (very important)

Inside Compose, **service names become hostnames**.

So if your DB service is called `mysql`, your backend connects like:

- host: `mysql`
- port: `3306`

This is why you should **not** use `localhost` from one container to talk to another container.

---

## 6) Environment variables (how it should be)

Environment variables keep configuration out of code and avoid hardcoding secrets.

In Compose you used:

- MySQL:
  - `MYSQL_ROOT_PASSWORD`
  - `MYSQL_DATABASE`
- Spring:
  - `SERVER_PORT`

For real deployment, the same idea applies:
- set DB credentials and URLs through environment variables in the hosting dashboard
- never commit production passwords into GitHub

---

## 7) Spring profiles (`dev` / `docker` / `prod`)

Profiles let the same codebase run safely in different environments:

- `dev` → local development (IDE)
- `docker` → docker-compose environment
- `prod` → cloud deployment (Railway/Render/Fly)

Common structure:
- `application.yml` (shared defaults)
- `application-dev.yml`
- `application-docker.yml`
- `application-prod.yml`

Activate with:
- `SPRING_PROFILES_ACTIVE=prod` (in production)
- `SPRING_PROFILES_ACTIVE=docker` (in compose)

---

## 8) Fullstack deployment (where each layer fits)

Different layers are deployed separately:

- **Frontend:** Vercel / Netlify / Cloudflare Pages (static hosting + CDN)
- **Backend:** Railway / Render / Fly / AWS (server/container hosting)
- **Database:** Railway DB / RDS / PlanetScale / Supabase / Mongo Atlas (managed DB)

Connection points:
- Frontend calls backend using the backend’s public URL (not `localhost`)
- Backend connects to database using **cloud DB variables** (not local compose hostnames)

---

## 9) Injecting environment variables into Spring config files

Spring config can reference environment variables directly.

Example `application-prod.yml`:

```yml
spring:
  datasource:
    url: jdbc:mysql://${DBHOST}:${DBPORT}/${DBDATABASE}
    username: ${DBUSER}
    password: ${DBPASSWORD}

  jpa:
    hibernate:
      ddl-auto: update
```

Your app reads these at runtime:
- `DBHOST`
- `DBPORT`
- `DBDATABASE`
- `DBUSER`
- `DBPASSWORD`

These values are typically provided by the hosting service (when you add a database).

### Default values (fallbacks)

Some values should have safe defaults:

```yml
server:
  port: ${PORT:8080}
```

Meaning:
- if `PORT` exists → use it
- otherwise → use `8080`

This is important because many platforms dynamically assign `PORT`.

### Local vs production workflow (profiles)

**Local**
- `SPRING_PROFILES_ACTIVE=dev`
- connects to local DB or docker-compose DB

**Production**
- `SPRING_PROFILES_ACTIVE=prod`
- connects to cloud DB via environment variables

Same codebase, different runtime configuration.

> Note: you typically **don’t** hardcode `spring.profiles.active` in YAML.
> Prefer setting it from the environment:
> - `SPRING_PROFILES_ACTIVE=prod` in the platform dashboard
> - `SPRING_PROFILES_ACTIVE=docker` in compose/Dockerfile

---

## 10) Deploying to Railway-like services (Railway / Render / Fly.io)

Most of these platforms follow the same pipeline:

**GitHub repo → build (Docker) → start service → inject env vars → app goes live**

### Step 1: Push code to GitHub
Ensure the repo includes:
- `Dockerfile`
- `pom.xml`
- (optional) `docker-compose.yml` for local development

### Step 2: Create a project + add a database
In Railway (example):
- New Project → Add MySQL
- Railway provides connection values in the DB/service dashboard.

### Step 3: Deploy the backend service from GitHub
- Create a new service from your GitHub repo.
- With a `Dockerfile`, the platform usually builds and runs it automatically.

### Step 4: Configure environment variables (service dashboard)
Set variables like:

- `SPRING_PROFILES_ACTIVE=prod`
- `DBHOST=...`
- `DBPORT=...`
- `DBDATABASE=...`
- `DBUSER=...`
- `DBPASSWORD=...`

And ensure your Spring config supports:
```yml
server:
  port: ${PORT:8080}
```

> If the platform provides different variable names (or a single `DATABASE_URL`), you can either:
> - change your `application-prod.yml` to match the platform variables, or
> - create your own variables in the dashboard (DBHOST/DBPORT/...) and copy values into them.

### Step 5: Deploy + verify
- Trigger deploy (or push to `main` if auto-deploy is enabled)
- Check logs for:
  - app started successfully
  - DB connection success
- Open the service public URL
- Test an API endpoint (or add a `/health` endpoint)

### Step 6: Connect the frontend
- Put the backend URL into frontend environment variables (example: `VITE_API_URL=...`)
- Deploy frontend to Vercel/Netlify/Cloudflare Pages

---

## Final architecture

### Local (development)
- Docker Compose:
  - Spring container
  - MySQL container

### Production (cloud)
- Backend container on Railway/Render/Fly
- Managed MySQL database
- Frontend deployed separately (Vercel/Netlify)

---
