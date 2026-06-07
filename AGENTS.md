# AGENTS.md

Guidance for cloud agents developing **MBCMS** (Multi-Branch Cinema Management System).

## Stack

- Java 17+ (Java 21 works), Maven 3.x, Apache Tomcat 10.1+, Microsoft SQL Server 2019+
- Monolithic Jakarta Servlet/JSP WAR — no Node.js/npm
- Frontend assets via CDN (Bootstrap, jQuery, Chart.js)

## Cursor Cloud specific instructions

### Services

| Service | Port | Notes |
|---------|------|-------|
| SQL Server (`mbcms-sqlserver` Docker container) | 1433 | Required for DB-backed features |
| Tomcat (`~/tomcat`) | 8080 | App context path: `/MBCMS` |

### First-time / after VM restart

1. **SQL Server** — start the container if it is stopped:
   ```bash
   sudo docker start mbcms-sqlserver
   ```
   SA password used in dev: `Mbcms_Dev@2026!` (database `CinemaDB`).

2. **database.properties** — must exist at `src/main/resources/database.properties` (gitignored). Copy from `database.properties.example` and set credentials. The app fails at deploy time if this file is missing.

3. **Build** — see `README.md` / `pom.xml`:
   ```bash
   mvn clean package
   ```

4. **Deploy & run Tomcat**:
   ```bash
   cp target/MBCMS.war ~/tomcat/webapps/
   export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
   export CATALINA_HOME=~/tomcat
   ~/tomcat/bin/catalina.sh run
   ```
   Visit: http://localhost:8080/MBCMS/home

### Database scripts

Run in order (idempotent): `database/CinemaDB_schema.sql` then `database/CinemaDB_seed.sql` via SSMS or:
```bash
sqlcmd -S localhost -U sa -P 'Mbcms_Dev@2026!' -C -i database/CinemaDB_schema.sql
sqlcmd -S localhost -U sa -P 'Mbcms_Dev@2026!' -C -i database/CinemaDB_seed.sql
```

Seed accounts (password `password`): `admin`, `mgr_hcm`, `staff_hcm`, `hungnt` — see `database/README.md`.

### Lint / test

- No dedicated linter configured. `mvn clean package` compiles all sources.
- JUnit/Mockito are declared; `src/test` may be empty — `mvn test` is a no-op until tests are added.

### Implementation status

Auth login POST, movie listing from DB, and several servlets are still TODO stubs. A successful deploy + HTTP 200 on `/home`, `/movies`, `/auth/login` confirms the environment; DB connectivity is verified when the WAR deploys without `database.properties` / pool errors in Tomcat logs.
