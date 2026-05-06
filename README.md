# Reddit Clone — Full-Stack REST API

A production-ready Reddit-style REST API built with Java Spring Boot, featuring JWT authentication, email verification, subreddit management, post/comment CRUD, and a complete upvote/downvote voting system. Deployed on Render with a Supabase PostgreSQL database.

**Live API:** [Swagger UI / Live API](https://reddit-clone-api.onrender.com/swagger-ui.html)
**GitHub:** [https://github.com/sukrutimallesh/Reddit-Clone](https://github.com/sukrutimallesh/Reddit-Clone)

> Backend API — Frontend available separately.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.1.1 |
| Security | Spring Security + JWT (stateless) |
| Persistence | Spring Data JPA + Hibernate |
| Database | PostgreSQL via Supabase |
| Email | Spring Mail (Gmail SMTP) |
| API Docs | Springdoc OpenAPI / Swagger UI |
| Containerization | Docker (multi-stage build) |
| Hosting | Render (Docker web service) |

---

## Architecture

The application follows a layered RESTful architecture:

```
Client Request
    └── Spring Security Filter Chain (JWT validation)
            └── @RestController (HTTP layer)
                    └── @Service (business logic)
                            └── @Repository / Spring Data JPA
                                        └── PostgreSQL (Supabase)
```

- **Stateless authentication**: Every request carries a signed JWT; no session state is stored server-side.
- **Entity relationships**: Users own Posts and Comments; Posts belong to Subreddits; Votes are tied to a User + Post pair, enforcing one vote per user per post.
- **Email verification**: On registration, a time-limited token is emailed to the user; the account is only activated after the link is clicked.
- **Auto DDL**: Hibernate manages the schema (`ddl-auto=update`), so tables are created/migrated automatically on startup.

---

## API Endpoints

### Authentication (`/api/auth`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/signup` | Register a new user (triggers verification email) |
| GET | `/api/auth/accountVerification/{token}` | Activate account via emailed token |
| POST | `/api/auth/login` | Login and receive a JWT |
| POST | `/api/auth/refresh/token` | Refresh an expired JWT |
| POST | `/api/auth/logout` | Invalidate refresh token |

### Subreddits (`/api/subreddit`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/subreddit` | Create a new subreddit |
| GET | `/api/subreddit` | List all subreddits |
| GET | `/api/subreddit/{id}` | Get a single subreddit by ID |

### Posts (`/api/posts`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/posts` | Create a post (authenticated) |
| GET | `/api/posts` | List all posts |
| GET | `/api/posts/{id}` | Get a single post |
| GET | `/api/posts/by-subreddit/{id}` | Get posts in a subreddit |
| GET | `/api/posts/by-user/{name}` | Get posts by a user |

### Comments (`/api/comments`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/comments` | Add a comment to a post (authenticated) |
| GET | `/api/comments/by-post/{postId}` | List comments on a post |
| GET | `/api/comments/by-user/{userName}` | List comments by a user |

### Votes (`/api/votes`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/votes` | Upvote or downvote a post (authenticated) |

---

## Features

- **JWT Authentication** — Secure, stateless token-based auth with configurable expiry
- **Email Verification** — Account activation flow via tokenized email links (Gmail SMTP)
- **Subreddits** — Create and browse topic-based communities
- **Posts** — Full CRUD with vote count tracking per post
- **Comments** — Nested discussion threads on posts
- **Upvote / Downvote** — Vote system that adjusts post scores in real time
- **Spring Security** — Method-level and route-level access control
- **Swagger UI** — Interactive API documentation at `/swagger-ui.html`
- **Actuator Health** — `/actuator/health` endpoint for Render health checks
- **Docker** — Multi-stage build for a minimal production image

---

## Security

JWT flow:
1. User sends credentials to `POST /api/auth/login`
2. Server validates credentials and returns a signed JWT + refresh token
3. Client includes the JWT in the `Authorization: Bearer <token>` header on every subsequent request
4. Spring Security's filter chain validates the token signature and expiry before passing the request to the controller
5. Refresh tokens allow obtaining new JWTs without re-entering credentials

Secrets (JWT secret, DB credentials, mail password) are injected via environment variables and never committed to source control.

---

## Local Setup

### Prerequisites
- Java 17
- Maven 3.8+
- PostgreSQL running locally (or a Supabase project)

### Steps

```bash
# Clone the repository
git clone https://github.com/sukrutimallesh/Reddit-Clone.git
cd Reddit-Clone

# Copy environment template
cp .env.example .env
# Edit .env with your local DB credentials and mail config

# Run the application
./mvnw spring-boot:run
```

The API starts on `http://localhost:8080`.
Swagger UI is available at `http://localhost:8080/swagger-ui.html`.

---

## Deployment Guide

### 1. Supabase (Database)

1. Create a free project at [supabase.com](https://supabase.com)
2. Go to **Project Settings > Database** and copy the connection string
3. (Optional) Run `supabase/schema.sql` in the Supabase SQL editor to pre-create tables

### 2. Render (App Hosting)

1. Push this repository to GitHub
2. Create a new **Web Service** on [render.com](https://render.com) and connect the repo
3. Render detects the `Dockerfile` and `render.yaml` automatically
4. Set the following environment variables in the Render dashboard:

| Variable | Description |
|---|---|
| `DATABASE_URL` | Full JDBC URL: `jdbc:postgresql://db.supabase.co:5432/postgres` |
| `DB_USERNAME` | Supabase DB username (usually `postgres`) |
| `DB_PASSWORD` | Supabase DB password |
| `JWT_SECRET` | Long random string (32+ chars) |
| `MAIL_USERNAME` | Gmail address for sending verification emails |
| `MAIL_PASSWORD` | Gmail App Password (not your account password) |

5. Deploy. Render will build the Docker image and start the service.
6. The health check hits `/actuator/health` — the service goes live once it returns `200 OK`.

### 3. Gmail App Password (for email verification)

1. Enable 2-Factor Authentication on your Google account
2. Go to **Google Account > Security > App Passwords**
3. Generate an app password for "Mail"
4. Use that 16-character password as `MAIL_PASSWORD`

---

## Project Structure

```
src/
  main/
    java/com/example/Reddit_Clone/
      config/          # Spring Security configuration
      controller/      # REST controllers
      dto/             # Request/Response DTOs
      model/           # JPA entities (User, Post, Comment, Vote, Subreddit, VerificationToken)
      repository/      # Spring Data JPA repositories
      security/        # JWT provider and filter
      service/         # Business logic
    resources/
      application.properties   # Environment-driven configuration
supabase/
  schema.sql           # Reference SQL schema
Dockerfile             # Multi-stage Docker build
render.yaml            # Render deployment config
```

---

## License

MIT
