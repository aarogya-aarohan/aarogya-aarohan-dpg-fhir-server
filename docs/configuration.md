# Configuration added by this fork

Everything here is set in `src/main/resources/application.yaml` under the `hapi.fhir` key, or through environment variables in Spring Boot form. For example, `hapi.fhir.binary_storage` becomes `HAPI_FHIR_BINARY_STORAGE`. Upstream HAPI settings are covered in the [HAPI starter guide](hapi-starter-guide.md).

## Authentication: `hapi.fhir.security.inbound.authentication`

| Key | Default | Meaning |
|---|---|---|
| `enabled` | `false` | Validate bearer tokens (OIDC JWT). Turn on in every real deployment. |
| `user-realm-uri` | none | Issuer URL of a single realm, for example `https://idp.example.org/realms/myrealm`. Used when `allowed-issuer-patterns` is empty. |
| `allowed-issuer-patterns` | empty | List of regular expressions matched against each token's `iss` claim, for one realm per tenant: `https://idp.example.org/realms/.*`. When set, `user-realm-uri` is ignored. |
| `proceed-to-authorization-on-failure` | `false` | Continue to the authorisation step when token validation fails. |
| `proceed-to-authorization-on-no-auth` | `false` (`application.yaml` sets `true`) | Continue to the authorisation step when there is no token, so anonymous rules apply. |

Signing keys are fetched from the issuer's standard OIDC discovery endpoints, so any OIDC provider works. Keycloak is the one we use.

## Authorisation: `hapi.fhir.security.inbound.authorization`

| Key | Default | Meaning |
|---|---|---|
| `enabled` | `false` | Enforce permissions on every request. |
| `allow-anonymous-access` | `true` | Allow requests without a token, limited to `anonymous-access-permission`. |
| `anonymous-access-permission` | see `application.yaml` | Permissions given to anonymous requests, each with a `name` and optional `arguments`. |

Permissions come from the token's `realm_access.roles` claim. A role whose name matches a permission grants it, for example `FHIR_ALL_READ`, `FHIR_ALL_WRITE`, `FHIR_TRANSACTION` or `FHIR_OP_ENCOUNTER_EVERYTHING`. Permissions that take an argument, such as `FHIR_READ_ALL_OF_TYPE`, read it from a `permission_argument` claim in the form `PERMISSION/argument`.

The full list is in `src/main/java/ca/uhn/fhir/jpa/starter/authnz/inbound/shared/Permission.java`. Names follow the Smile CDR permission model.

## Multi-tenancy: `hapi.fhir.partitioning`

Partitioning is off unless this block is present. When it is on:

- Tenants are addressed by URL: `/fhir/{TENANT}/Patient`.
- Requests without a tenant segment, or with `/fhir/DEFAULT/...`, go to the shared `DEFAULT` partition.
- For any other tenant, an authenticated token must carry a `partition_id` claim equal to the tenant name. Requests with no claim, or a different one, are rejected.

| Key | Default | Meaning |
|---|---|---|
| `allow_references_across_partitions` | `true` | Let tenant resources reference shared definitional resources (Questionnaire, ValueSet and so on) that live in `DEFAULT`. |
| `partitioning_include_in_search_hashes` | `false` | Upstream HAPI setting. |
| `default_only_resource_types` | empty | Resource types always read from `DEFAULT`, even through a tenant URL (for example Composition, StructureMap, PlanDefinition, Library). Writes are not redirected. |
| `merge_default_resource_types` | empty | Resource types whose reads return the tenant's own resources plus those in `DEFAULT` (for example Location). Writes stay in the tenant. |

To add a tenant: create its partition, create its identity realm (or client) so that tokens carry `partition_id: <TENANT>`, and add its issuer to `allowed-issuer-patterns` if it has its own realm.

## Image (binary) storage

| Key | Default | Meaning |
|---|---|---|
| `binary_storage_enabled` | `false` | Store large binaries outside the resource table. |
| `binary_storage` | `Database` | Backend: `Database`, `Minio` or `Gcs`. |
| `max_binary_size` | upstream default | Largest accepted binary, in bytes. |

### `Minio`: any S3-compatible store

Works with MinIO, AWS S3, and other stores that speak the S3 API.

| Key under `binary_storage_minio` | Meaning |
|---|---|
| `base-url` | S3 API endpoint, for example `https://s3.example.org`. |
| `access-key`, `secret-key` | Credentials. Supply them from a secret store or environment variables, never from a committed file. |
| `bucket-name` | Bucket for all binaries. |
| `base-path` | Optional prefix inside the bucket. |

All tenants share the one bucket.

### `Gcs`: Google Cloud Storage, one bucket per tenant

| Key under `binary_storage_gcs` | Meaning |
|---|---|
| `bucket_name_template` | Bucket name with a `{partition}` placeholder, for example `myorg-{partition}-fhir-media`. Required. |
| `base_path` | Optional prefix inside each bucket. |

Credentials come from Google Application Default Credentials (workload identity or a mounted key). Writes to the `DEFAULT` partition are refused; use a tenant URL.

## Database

PostgreSQL is used in production. Set:

```
SPRING_DATASOURCE_URL=jdbc:postgresql://<host>:5432/<db>
SPRING_DATASOURCE_USERNAME=<user>
SPRING_DATASOURCE_PASSWORD=<from your secret store>
SPRING_DATASOURCE_DRIVERCLASSNAME=org.postgresql.Driver
SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT=ca.uhn.fhir.jpa.model.dialect.HapiFhirPostgresDialect
```

On Google Cloud SQL you can use the Cloud SQL proxy as a sidecar, or build with the optional `cloudsql-postgres` Maven profile.

## Metrics

`/actuator/prometheus` exposes Micrometer metrics, including per-endpoint request counters by resource type, operation and status class. The counters are registered through `custom-bean-packages` and `custom-interceptor-classes` in `application.yaml`.
