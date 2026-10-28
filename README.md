# Rankaku

Monorepo containing:

- `apps/web`: Angular web application
- `apps/api`: Spring Boot (Java 17) API

## Prerequisites

- Node.js 20+
- Java 17+
- Maven 3.9+

## Web app (Angular)

```bash
cd /home/runner/work/Rankaku/Rankaku/apps/web
npm install
npm run start
```

## API (Spring Boot)

```bash
cd /home/runner/work/Rankaku/Rankaku/apps/api
mvn spring-boot:run
```

Health endpoint:

```bash
curl http://localhost:8080/api/health
```

## Build & test

```bash
cd /home/runner/work/Rankaku/Rankaku/apps/api
mvn test
```

```bash
cd /home/runner/work/Rankaku/Rankaku/apps/web
npm run build
```