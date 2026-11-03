# Inventory Management System: Spring Boot + DevOps pipeline

Made during the IBM DevOps, Agile & Design Thinking course (June–July 2025). A Spring Boot REST API for managing inventory, wrapped in a DevOps setup: a Jenkins pipeline, a multi-stage Docker build, an Ansible deploy stage, and metrics exported to Graphite and viewed in Grafana.

## What's in the repo

| Part | What it does | Files |
|---|---|---|
| REST API | CRUD for products, search by name and category, low-stock and out-of-stock queries, and inventory stats | `src/main/java/com/devops/inventory/` |
| Database | In-memory H2, seeded from `data.sql` on startup (H2 console at `/h2-console`) | `src/main/resources/` |
| API docs | Swagger UI via springdoc-openapi | `/swagger-ui.html` |
| Monitoring | Spring Boot Actuator (health, metrics, Prometheus endpoint) and Micrometer metrics pushed to Graphite every 10 s | `application.yml` |
| Container | Multi-stage Dockerfile (Maven build, then a slim JRE image running as a non-root user) | `Dockerfile` |
| Local stack | The app plus Graphite, Grafana and Jenkins in one `docker compose` setup | `docker-compose.yml` |
| CI/CD | Jenkins pipeline: Checkout → Build → Test → Package → Docker Build → Deploy with Ansible | `Jenkinsfile`, `ansible/` |
| Setup | A helper script that checks for and installs the tools above on Ubuntu | `setup_devops_project.sh` |

## API

Base path: `/api/products`

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | List all products |
| GET | `/{id}` | Get one product |
| GET | `/name/{name}` | Search by name |
| GET | `/category/{category}` | Filter by category |
| GET | `/low-stock/{threshold}` | Products below a stock threshold |
| GET | `/out-of-stock` | Products with zero stock |
| GET | `/stats/in-stock-count` | Number of products in stock |
| GET | `/stats/total-inventory` | Total units in stock (sum of all quantities) |
| POST | `/` | Create a product |
| PUT | `/{id}` | Update a product |
| DELETE | `/{id}` | Delete a product |

## Run it

With Docker:

```bash
docker compose up --build
```

| Service | URL |
|---|---|
| App | http://localhost:8081 (Swagger UI at `/swagger-ui.html`) |
| Grafana | http://localhost:3000 |
| Graphite | http://localhost:8082 |
| Jenkins | http://localhost:8080 |

Without Docker (Java 11 and Maven needed):

```bash
mvn spring-boot:run
```

The app then runs on http://localhost:8080.

## Known gaps

I'm listing these honestly so the repo matches what it actually does:

- **No unit tests yet.** The pipeline has a Test stage and JUnit 5 is a dependency, but `src/test` is empty.
- **The Ansible deploy is a placeholder.** `ansible/playbook.yml` only prints a message to show Jenkins can trigger Ansible; it doesn't deploy the container yet.
- **Inventory file name mismatch.** The Jenkinsfile expects `ansible/inventory.ini`, but the file is named `inventory.imi`, so the Ansible stage needs that fixed before it can run.
- **In-memory database.** Data resets on every restart; a real deployment would need PostgreSQL or MySQL.

## Tech

Java 11, Spring Boot (Web, Data JPA, Validation, Actuator), H2, springdoc-openapi, Micrometer, Maven, Docker, Docker Compose, Jenkins, Ansible, Graphite, Grafana
