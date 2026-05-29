# ReviewPilot AI

GitHub App that reviews Pull Requests using Java, Spring Boot and LLMs.

## Demo
Still in desenvolviment...

## Why I built this
I was manually pasting diffs into LLMs before opening PRs. I automated the workflow.

## Architecture
Diagram.

## Features
- GitHub webhook signature validation
- Pull Request diff fetching
- LLM-based review generation
- Idempotency by commit SHA
- PostgreSQL persistence
- Dockerized local environment
- CI/CD

## Tech Stack
Java 21, Spring Boot 3, PostgreSQL, Docker, GitHub Apps, LLM API.

## How it works
1. GitHub sends webhook
2. API validates signature
3. Worker fetches diff
4. Diff is chunked
5. LLM reviews
6. Bot comments on PR

## Running locally

Start the local PostgreSQL database:

```bash
docker compose up -d
```

Run the application with the `local` profile:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

Check the health endpoint:

```bash
curl http://localhost:8080/actuator/health
```

## Testing

Run the automated test suite:

```bash
./mvnw test
```

Tests use the `test` profile with an in-memory H2 database, so they do not require a local PostgreSQL instance.

## Roadmap
MVP, v0.2, v1.0.

## Lessons learned
Context limits, cost control, webhook security, idempotency.
