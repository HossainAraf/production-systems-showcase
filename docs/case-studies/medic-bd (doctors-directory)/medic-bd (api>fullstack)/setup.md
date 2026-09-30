# Setup guide

This project uses Ruby on Rails with PostgreSQL. The setup below is intentionally generic so it can be reused in public-facing documentation without exposing local environment details.

## Prerequisites

- Ruby (matching the project version)
- Bundler
- PostgreSQL installed and running
- A local database user with create privileges

## Install dependencies

```bash
bundle install
```

## Create the databases

```bash
bin/rails db:create
```

For the test database, use:

```bash
RAILS_ENV=test bin/rails db:create
```

## Run migrations

```bash
bin/rails db:migrate
```

For tests:

```bash
RAILS_ENV=test bin/rails db:migrate
```

## Start the app

Use the development runner:

```bash
bin/dev
```

This starts the Rails server and the asset watcher together.

## Verify the app

Open the local app in a browser or call the health endpoint:

```bash
curl http://localhost:3000/health
```

Expected response:

```text
OK
```

## Optional checks

Create a schema dump if needed:

```bash
bin/rails db:schema:dump
```

Run the test suite:

```bash
bundle exec rails test
```

Run RuboCop:

```bash
bundle exec rubocop
```

## Notes

- Keep credentials and secret values in environment variables or Rails credentials, never in shared documentation.
- Use the project’s configured database adapter and search path settings from the Rails app configuration.
- If the app fails to boot, verify that PostgreSQL is available and the database has been created before running migrations.

