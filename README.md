# Aarogya Aarohan FHIR server

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

The clinical data store for **Aarogya Aarohan**, an oral-cancer screening platform. It is a FHIR R4 server built on [HAPI FHIR](https://hapifhir.io/) 7.2.0, forked from the [HAPI FHIR JPA Server Starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter).

The Android app and the FHIR gateway store patients, encounters, questionnaire responses, tasks and screening images here. The web dashboard reads them back through the gateway.

> **Clinical disclaimer.** Aarogya Aarohan is an **assisted screening tool**. It is **not an autonomous diagnostic device**; clinical decisions remain with qualified professionals.

---

## What this fork adds to the upstream starter

| Feature | What it does | Code (under `src/main/java/ca/uhn/fhir/jpa/starter/`) |
|---|---|---|
| Multi-tenancy | One FHIR partition per tenant, addressed by URL (`/fhir/{TENANT}/Patient`). Shared definitional resources stay in `DEFAULT`. | `common/StarterJpaConfig.java`, `authnz/inbound/authorization/SystemAwareRequestTenantPartitionInterceptor.java` |
| Tenant isolation by token | Each tenant's token must carry a `partition_id` claim matching the tenant in the URL. Mismatches are rejected. | `authnz/inbound/authorization/JwtPartitionValidationInterceptor.java` |
| OIDC authentication | Validates JWTs from one issuer, or from several issuers matched by pattern (one Keycloak realm per tenant). | `authnz/inbound/authentication/` |
| Permission-based authorisation | Maps token roles to FHIR permissions (read, write, transaction, operations), with configurable anonymous access. | `authnz/inbound/authorization/` |
| Pluggable image storage | Screening images go to the database, an S3-compatible store (MinIO, AWS S3 and similar), or Google Cloud Storage with one bucket per tenant. | `binary/`, `common/FhirServerConfigCommon.java` |
| OpenSRP-style operations | `PractitionerDetail` and `LocationHierarchy` endpoints used by the Android app. | `opensrp/` |
| Per-endpoint metrics | Prometheus counters by resource type, operation and status class, at `/actuator/prometheus`. | `observability/FhirEndpointMetricsInterceptor.java` |

No proprietary service is required. PostgreSQL and any OIDC provider (we use Keycloak) are enough, plus any S3-compatible object store if images are kept outside the database. Google Cloud Storage and the Cloud SQL connector are optional.

---

## Quick start (local)

Requirements: Docker with Compose.

```bash
cp .env.example .env        # set POSTGRES_PASSWORD
docker compose up --build
curl -s http://localhost:8080/fhir/metadata | head
```

This starts the server with PostgreSQL 16, single-tenant, authentication off. The FHIR base URL is `http://localhost:8080/fhir`, and a web tester runs at `http://localhost:8080/`.

### Build without Docker

Requirements: JDK 17 and Maven 3.9.

```bash
mvn clean package -DskipTests
mvn spring-boot:run -Pboot
```

This uses an in-memory H2 database by default. Point it at PostgreSQL with the `SPRING_DATASOURCE_*` variables shown in `docker-compose.yml`.

### Tests

```bash
mvn verify                       # unit and integration tests
scripts/test-multi-tenant.sh     # end-to-end tenant isolation test (starts Keycloak in Docker)
```

---

## Configuration

All settings live in [`src/main/resources/application.yaml`](src/main/resources/application.yaml). Any of them can be overridden by environment variables in Spring Boot form, for example `HAPI_FHIR_BINARY_STORAGE=Minio`.

The settings this fork adds are described in [docs/configuration.md](docs/configuration.md). Upstream settings are described in the [HAPI starter guide](docs/hapi-starter-guide.md).

## Container images

`Dockerfile` builds two targets:

- `default`: distroless Java 17, non-root. This is the one to run.
- `tomcat`: Tomcat with a shell, for debugging only.

Pushing a tag `image/vX.Y.Z` builds both and publishes them to this repository's GitHub Container Registry (`.github/workflows/build-images.yaml`).

## Deploying

This server is one part of the platform. The infrastructure repository, `aarogya-aarohan-dpg-infra`, deploys it with the other services. It can run on a managed Kubernetes cloud with multiple tenants, or as a self-hosted stack per site.

Minimum production settings:

- PostgreSQL (16 recommended) with `HapiFhirPostgresDialect`.
- `security.inbound.authentication.enabled: true` with your OIDC issuer, and `security.inbound.authorization.enabled: true`.
- Images in object storage rather than the database once volumes grow.
- Liveness probe timeouts of at least 10 seconds, because HAPI can pause under load.

## Known limitations

- The S3/MinIO backend writes all tenants to one bucket. Only the Google Cloud Storage backend gives each tenant its own bucket. A partition-aware S3 backend is planned.
- With the GCS backend, images cannot be stored in the `DEFAULT` partition. They must be written through a tenant URL.

## Contributing and security

- [CONTRIBUTING.md](CONTRIBUTING.md)
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## License and attribution

Licensed under the [Apache License 2.0](LICENSE).

This project is a fork of the [HAPI FHIR JPA Server Starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter) by the HAPI FHIR project, also Apache 2.0.
