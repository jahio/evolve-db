# Evolve: A standalone tool for SQL database migrations

Say you want to be able to have the convenience of database migrations in multiple projects that don't share the same language, framework or codebase,
but don't want to rely on writing raw SQL scripts to do that.

The point of Evolve is to let you define your database schema in YAML, then run the Evolve CLI (`evolve-db`) against them, and it will figure out the
difference between the current state of the database and the state it should be in as defined at runtime by your YAML, then apply the necessary changes.

With Evolve, you don't keep track of individual files, each representing an individual change to be applied one-by-one. Instead, you keep ONE plain
text YAML file per table in your database. That's it. And then as your application's needs change over time, you simply update that file and re-run
Evolve against it again. Evolve will figure out what needs to change on its own, and then handle those changes for you automatically.

## Quick Example of DSL/YAML

Say you'd like to define a users table for your blog app:

```yaml
---
# users.yaml
table: users
primary_key:
  - type: uuid
    name: id
created_at: true
updated_at: true
columns:
  - name: name
    type: string
    null: false
  - name: email
    type: string
    null: false
indexes:
  - name: idx_users_email
    columns:
      - email
    unique: true
```

And then of course you need the table that holds the articles:

```yaml
---
# articles.yaml
table: articles
primary_key:
  - type: uuid
    name: id
created_at: true
updated_at: true
columns:
  - name: title
    type: string
    null: false
  - name: subtitle
    type: string
  - name: body
    type: text
  - name: user_id
    type: uuid
    fk:
      table: users
      column: id
indexes:
  - name: idx_articles_title_user_id
    columns:
      - title
      - user_id
    unique: true
    
```

## MVP Features

- [ ] Support for PostgreSQL
- [ ] Creation of tables and columns with optional indexes as defined in YAML
- [ ] Simple foreign-key support
- [ ] Schema export into YAML, SQL, JSON, and XML
- [ ] Behavior configuration via CLI options or environment variables
- [ ] Support for a "plan" feature that dumps the proposed changes on screen and lets you decide whether or not to continue
- [ ] Support for an override for that "plan" feature so that in production/deployment scenarios you can force it right through, no waiting on STDIN
- [ ] Support for allowing the user to write raw SQL within the YAML should they need to in whatever edge cases that may pop up
- [ ] Support for allowing the user to define explilcit "up" and "down" sections if they absolutely have to
- [ ] Evolve should be able to figure out its own execution order based on the dependencies of the tables and columns defined in the YAML so it doesn't try to create a foreign key for a table that doesn't exist yet
- [ ] Combination indexes (e.g. `UNIQUE (column1, column2)`)
- [ ] Automatically concoct an appropriate index name if not specified
- [ ] Allow user to manually override that by specifying their own index name in YAML

## Future Features

- [ ] Support for MySQL, SQLite
- [ ] Ability to specify seed data within the YAML itself
- [ ] ...or to add a command to the YAML that can be invoked to create the seed data after the table is created
- [ ] ...with an option to delay that command until the entire database schema has been fully migrated up
- [ ] Automatic translation between incompatible datatypes between databases (e.g. `TEXT` in SQLite to `VARCHAR` in PostgreSQL)
- [ ] Warning when relying on a data type not available in other SQL databases (e.g. `uuid` or `json` in PostgreSQL)
- [ ] Ability to suggest - and automatically apply, in deployment - a suitable automatic replacement for incompatible datatypes (e.g. `uuid` in psql -> `varchar(36)` in MySQL)
- [ ] Support for specifying the limit/number of characters in a varchar column
- [ ] Support for Support for specifying the precision and scale of a decimal column
- [ ] Before and after hooks for the entire migration process
- [ ] Before and after hooks for each individual table migration within the process
