# Environment Variables Setup (Spring + React)

## 0) Core Idea

Environment variables are **external configuration**:

```env
KEY=VALUE
```

Common uses:

- Database configuration
- API keys
- Ports
- Secrets

---

## 1) Spring Boot — Development (localhost)

### Option 1: Set variables in the terminal

Windows (PowerShell):

```powershell
$env:DB_URL="jdbc:mysql://localhost:3306/mydb"
$env:DB_USER="root"
$env:DB_PASS="1234"

mvn spring-boot:run
```

### Option 2: IntelliJ IDEA (practical for local dev)

Go to:

`Run → Edit Configurations → Environment variables`
If you don't see input field for env variable in your current spring config, create a new **application config**.

Add (in .env and export to config or set manually):

```env
DB_URL=jdbc:mysql://localhost:3306/mydb
DB_USER=root
DB_PASS=1234
```

### Access variables in Spring Boot

#### `application.properties`

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```

#### Using `@Value`

```java
@Value("${DB_URL}")
private String dbUrl;
```

#### Using `@ConfigurationProperties` (recommended as config grows)

Use this approach when you have many related settings and want structured configuration binding.

---

## 2) React — Development (localhost)

### Use a `.env` file

Create:

```env
.env
```

### Important rule

React or Vite only exposes environment variables that start with:

```env
CONSTANT = REACT_APP_ || VITE_
```

Example:

```env
${CONSTANT}_API_URL=http://localhost:8080/api
```

Usage in code:

```javascript
const api = process.env.${CONSTANT}_API_URL;
```

### Restart required

After changing `.env`, restart the dev server:

```bash
npm start
```

React does not hot-reload environment variable changes.

---

## 3) Spring Boot — Docker

### Method 1: `docker-compose.yml` with `environment`

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_URL: jdbc:mysql://mysql:3306/mydb
      DB_USER: root
      DB_PASS: 1234
```

### Method 2: `.env` file with Docker Compose

Create a `.env` file:

```env
DB_URL=jdbc:mysql://mysql:3306/mydb
DB_USER=root
DB_PASS=1234
```

Then reference it in `docker-compose.yml`:

```yaml
env_file:
  - .env
```

### Method 3: Dockerfile `ENV` (not recommended for secrets)

```dockerfile
ENV DB_URL=jdbc:mysql://...
```

This is not recommended for secrets because the values get baked into the image.

### Method 4: Runtime injection with `docker run`

```bash
docker run -e DB_URL=... -e DB_USER=root -e DB_PASS=1234 your-image
```

### Note

Spring Boot can read environment variables directly:

- `DB_URL` maps to `${DB_URL}`

No additional setup is required.

---

## 4) Deployment (Spring Boot + React)

This is where configuration is commonly misunderstood.

### I. Spring Boot deployment

On platforms like Render, Railway, AWS, or Heroku, you set environment variables in the platform dashboard, for example:

```env
DB_URL=...
DB_USER=...
PORT=10000
```

#### PORT handling (important)

Configure Spring Boot to use the hosting platform’s port while keeping a local default:

```properties
server.port=${PORT:8080}
```

Meaning:

- Use `PORT` when provided by the host
- Default to `8080` locally


### II. React deployment

On hosting platforms (Vercel, Netlify, Render), you set the environment variables in their dashboard. React environment variables are **baked in at build time**.

Example `.env.production`:

```env
REACT_APP_API_URL=https://your-api.com
```

The key point is: **changing them requires rebuilding/redeploying**.

---

## 5) Common Mistakes

1. **Forgetting `REACT_APP_`**  
   React will not expose variables without the required prefix.

2. **Hardcoding secrets in code**  
   Example (bad):
   ```java
   String password = "1234";
   ```

3. **Not using `PORT` in deployment**  
   Can lead to startup failures on platforms that expect the app to bind to a specific port.

4. **Committing `.env` files**  
   Add `.env` to `.gitignore` to avoid leaking secrets.
