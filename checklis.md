## 2026-05-26

### Project setup

- [x] Defined the MVP scope for ReviewPilot AI
- [x] Created the GitHub repository
- [x] Created the Spring Boot project
- [x] Configured the project with Java 21, Spring Boot 3.5.14, Maven, and Jar packaging
- [x] Set the base package to `com.enzoviniski.reviewpilotai`
- [x] Added initial dependencies: Spring Web, Spring Security, Spring Data JPA, PostgreSQL Driver, Flyway, Validation, Actuator, Lombok, and Testcontainers
- [x] Added the roadmap Markdown file to the project
- [x] Created the project folder/drive organization
- [x] Connected the local project to the GitHub repository
- [x] Committed the initial project setup
- [x] Pushed the initial commit to GitHub
- [x] Created the 10 initial GitHub issues for the MVP roadmap

### Validation

- [x] Verified that Maven uses Java 21
- [x] Ran the initial Maven test command
- [x] Confirmed that the main application code compiles
- [x] Identified that tests fail because the PostgreSQL datasource is not configured yet
- [x] Confirmed that Docker Compose with PostgreSQL is the next required task

### Next task

- [x] Add Docker Compose with PostgreSQL
- [x] Configure the local Spring profile
- [x] Run the application with the `local` profile
- [x] Verify that `/actuator/health` returns `UP`

## 2026-05-28

### Local PostgreSQL setup

- [x] Created `docker-compose.yml` for local PostgreSQL
- [x] Configured PostgreSQL database name, user, and password
- [x] Configured PostgreSQL data persistence using a Docker volume
- [x] Resolved the local port conflict by mapping host port `5433` to container port `5432`
- [x] Created the `local` Spring profile
- [x] Configured the Spring datasource for the `local` profile
- [x] Started PostgreSQL with Docker Compose
- [x] Ran the Spring Boot application using the `local` profile
- [x] Verified that `/actuator/health` returns `UP`

### Debugging notes

- [x] Identified a Java runtime mismatch between Java 21 and Java 17
- [x] Confirmed that class file version `65.0` means Java 21
- [x] Confirmed that class file version `61.0` means Java 17
- [x] Fixed the runtime environment so Maven runs with Java 21
- [x] Identified that PostgreSQL uses port `5432` inside the container
- [x] Learned that port `3306` is commonly used by MySQL, not PostgreSQL
- [x] Fixed the Docker port mapping from `3306:3306` to `5433:5432`

### Validation

- [x] `docker compose up -d`
- [x] `mvn clean spring-boot:run -Dspring-boot.run.profiles=local`
- [x] `curl http://localhost:8080/actuator/health`
- [x] Health check returned `UP`

### GitHub progress

- [x] Completed issue: `Bootstrap Spring Boot project`
- [x] Completed issue: `Add Docker Compose with PostgreSQL`

### Next task

- [x] Commit and push the local PostgreSQL setup
- [x] Open a pull request
- [x] Merge the pull request into `main`
- [x] Close issues 1 and 2 after merge
- [x] Start issue 3: `Configure application profiles`

## 2026-05-29

### Application profiles

- [x] Switched work to branch `chore/configure-application-profiles`
- [x] Reviewed the default `application.properties`
- [x] Kept local PostgreSQL settings in `application-local.properties`
- [x] Created `application-test.properties`
- [x] Configured automated tests to use the `test` profile
- [x] Added H2 as an in-memory database for tests
- [x] Configured H2 dependency with test scope
- [x] Disabled SQL script initialization in tests with `spring.sql.init.mode=never`
- [x] Removed `data.sql` because it was being loaded automatically and breaking the test context

### Validation

- [x] Ran `./mvnw test`
- [x] Confirmed tests do not require manually running PostgreSQL

### Documentation

- [x] Documented how to run the application with the `local` profile
- [x] Documented how to run the automated tests

### Next task

- [ ] Review the final diff before committing
- [ ] Commit the profile configuration changes
- [ ] Push the branch
- [ ] Open a pull request
- [ ] Merge the pull request into `main`
- [ ] Close issue 3 after merge
- [ ] Prepare issue 4: `Create GitHub webhook endpoint`
