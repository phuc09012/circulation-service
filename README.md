# CirculationService Package

## What is included

- Source for `CirculationService`
- Shared contracts used by the service
- External database script: `database/CirculationDb.sql`

## Connection contract

This repo depends on the other 2 service repos. Keep these shared values identical in all services:

- `Jwt__Issuer` = `LibraryAuth`
- `Jwt__Audience` = `LibraryUsers`
- `Jwt__Key` = same secret in every service
- `InternalApi__Key` = same internal secret in every service

This service must also be able to reach:

- `CatalogService`
- `IdentityReportService`

Default Docker Compose addresses:

- `CatalogService__BaseUrl=http://catalogservice:8080`
- `IdentityService__BaseUrl=http://identityreportservice:8080`
- `book.borrowed` / `book.returned` / `fine.paid` events -> `http://identityreportservice:8080/integration/events/...`

If the teams run on different machines, replace `localhost` and `*.service:8080` with real hostnames or IPs that are reachable from this service.

## Database setup

1. Open SQL Server Management Studio or Azure Data Studio.
2. Connect to your SQL Server instance.
3. Run `database/CirculationDb.sql`.
4. Verify the database name matches the service connection string.

## Default connection string

```json
Server=localhost,1433;Database=CirculationDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True;Encrypt=False
```

## Required environment variables

If you run this service outside the main compose stack, set these values explicitly:

```bash
ConnectionStrings__CirculationDb=Server=YOUR_SQL_SERVER;Database=CirculationDb;User Id=sa;Password=...;TrustServerCertificate=True;Encrypt=False
Jwt__Issuer=LibraryAuth
Jwt__Audience=LibraryUsers
Jwt__Key=ChangeThisKeyToSomethingAtLeast32CharsLong!
InternalApi__Key=LibraryInternalSecretChangeMe!
CatalogService__BaseUrl=http://YOUR_CATALOG_HOST:5001
IdentityService__BaseUrl=http://YOUR_IDENTITY_HOST:5003
IntegrationEvents__Subscribers__book.borrowed__0=http://YOUR_IDENTITY_HOST:5003/integration/events/book-borrowed
IntegrationEvents__Subscribers__book.returned__0=http://YOUR_IDENTITY_HOST:5003/integration/events/book-returned
IntegrationEvents__Subscribers__fine.paid__0=http://YOUR_IDENTITY_HOST:5003/integration/events/fine-paid
```

## Run locally

```bash
dotnet restore
dotnet run --project src/CirculationService/CirculationService.csproj
```

## Run with Docker

```bash
docker build -f src/CirculationService/Dockerfile -t circulationservice .
docker run --rm -p 5002:8080 circulationservice
```

## Notes

- This service needs CatalogService reachable at `CatalogService__BaseUrl`.
- When the whole system runs, it publishes borrow/return events to IdentityReportService.
- If `CatalogService` or `IdentityReportService` is moved to another host, update the URLs above before testing borrowing, return, or fine payment flows.
- Keep JWT and `InternalApi__Key` synchronized with the other repos or protected endpoints will reject calls.
