# PostgREST

## Setup PostgreSQL Database for PostgREST API

1. Create role to use for anonymous web requests and a dedicated role for connecting to the database, instead of using the highly privileged `postgres` role:

   ```SQL
   CREATE ROLE web_anon NOLOGIN;
   CREATE ROLE authenticator NOINHERIT LOGIN PASSWORD 'mysecretpassword';
   GRANT web_anon TO authenticator;
   ```

2. Setup specific API schema:

   ```SQL
   CREATE SCHEMA api;
   GRANT USAGE ON SCHEMA api TO web_anon;
   GRANT SELECT ON ALL TABLES IN SCHEMA api TO web_anon;
   ALTER DEFAULT PRIVILEGES IN SCHEMA api GRANT SELECT ON TABLES TO web_anon;
   ```

3. PostgREST configuration:

   ```bash
   PGRST_DB_URI: "postgres://authenticator:mysecretpassword@db.staging.mapx.org:5432/mapx_staging"
   PGRST_DB_SCHEMAS = "api"
   PGRST_DB_ANON_ROLE = "web_anon"
   ```