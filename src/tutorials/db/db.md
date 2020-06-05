# Introduction

In these tutorials we will show you a way to document your database using Foliant. Right now there are options available for those of you who use **PostgreSQL**, **Oracle** and **DBML**, but we will probably add more vendors later.

## The principles

Generally, we want to keep our docs as close to the code as possible. When documenting source code we usually can utilize the power of Swagger to generate docs from comments in the sources. Since there's no Swagger for databases, we had to invent something similar.

We are going to add actual descriptions of the tables and fields using *comments*. Comment (not to be confused with SQL script `--comments`) is a special entity which in one way or another is present in most DBMSs. They don't affect the data or table structure, they are only used for documentation purposes. You can add a comment like this:

```sql
COMMENT ON TABLE "Clients" IS "Table holding info about the clients"
```

After describing all your entities inside your database, we need to get all this data somehow. For this, we will use Foliant [DBDoc preprocessor](https://foliant-docs.github.io/docs/preprocessors/dbdoc/). It queries the database to get its structure (including our comments) and converts it into Markdown. We can then use this Markdown to generate a static site for our documentation.

## The tutorials

[Documenting with DBML specification](dbml.md)

[Documenting Oracle Database](oracle.md)

[Documenting PostgreSQL Database](pgsql.md)
