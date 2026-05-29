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

- [ ] Add Docker Compose with PostgreSQL
- [ ] Configure the local Spring profile
- [ ] Run the application with the `local` profile
- [ ] Verify that `/actuator/health` returns `UP`