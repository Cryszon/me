# Environment variables and build args for Docker Compose in Coolify

When you define environment variables in Coolify UI, Coolify creates a `.env`
file with those variables next to the `docker-compose.yml` file during
deployment.

This means that you can use any environment variable defined in the UI in your
`docker-compose.yml` file as long as **Build Variable?** is not checked.

## How to use build args with Docker Compose

If you want to define a build arg, just add it as an environment
variable in Coolfiy UI and use it in `docker-compose-yml`.

> [!IMPORTANT]
>
> Don't use **Build Variable?** for build args when using Compose. They don't
> work.

```yml
# docker-compose.yml
services:
  app:
    build:
      dockerfile: Dockerfile
      args:
        - TEST_SECRET
```

By omitting the value of a build arg, Docker Compose automatically uses the
value defined in the `.env` file written by Coolify.

## Env variables with quotes (e.g. JSON)

To use environment variables with quotes, set them in **Developer View**, switch
back to normal view and check **Is Literal?**.

Example of env variable with quotes:

```ini
COMPOSER_AUTH_JSON='{ "github-oauth": { "github.com": "github_pat_XXXX" } }'
```

## Updating env variables

The best way to ensure env variables are updated is to delete and recreate them.
Once deleted, you can add them again in **Developer view**, switch back to
normal view and set options (such as **Is Literal?**) and optionally lock them.

> [!CAUTION]
>
> Locked environment variables are only hidden in the Coolify UI. They are still
> stored in plain text on the host system in
> `/data/coolify/applications/<id>/.env`.
