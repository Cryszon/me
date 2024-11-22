# Docker build secrets in Coolify

> [!CAUTION]
>
> This article has been migrated to [cryszon.github.io](cryszon.github.io). This
> version might not be up to date.

<details>
<summary>Show article</summary>

Coolify doesn't currently _(2024-08-20)_ officially support build secrets. There
is a [discussion on
Discord](https://discord.com/channels/459365938081431553/1273315947893096470)
with research on the topic. In some cases build secrets might appear to work,
but they are unreliable.

Here's a workaround using build args insead of secrets.

> [!CAUTION]
>
> [Build variables are not a secure replacement for
> secrets](https://docs.docker.com/build/building/variables/)! Using them makes
> the secret visible in the final built image. `sh -c '...'` is used to hide the
> secret from the build logs and the temp file is removed (`rm /tmp/...`), but
> the secret value will still be readable by inspecting the built image.
>
> Environment variables are also stored in plain text in
> `/data/coolify/applications/<id>/.env` on the host system.
>
> See also [this
> documentation](https://docs.docker.com/reference/dockerfile/#arg) and [this
> discussion](https://stackoverflow.com/q/44615837/1865857).

Let's say you have following files:

```yml
# docker-compose.yml
services:
  app:
    build:
      dockerfile: Dockerfile
      secrets:
        - TEST_SECRET

secrets:
  TEST_SECRET:
    environment: TEST_SECRET
```

```Dockerfile
# Dockerfile
FROM alpine:latest

RUN --mount=type=secret,id=TEST_SECRET \
  cat /run/secrets/TEST_SECRET > /test_secret_file

ENTRYPOINT ["tail", "-f", "/dev/null"]
```

Here's how you use build args instead:

```yml
# docker-compose.yml
services:
  app:
    build:
      dockerfile: Dockerfile
      args:
        - TEST_SECRET
```

```Dockerfile
# Dockerfile
FROM alpine:latest

ARG TEST_SECRET
RUN sh -c 'echo "${TEST_SECRET}" > /tmp/TEST_SECRET' && \
  cat /tmp/TEST_SECRET > /test_secret_file
RUN unset TEST_SECRET && rm /tmp/TEST_SECRET

ENTRYPOINT ["tail", "-f", "/dev/null"]
```

See [this
article](environment-variables-and-build-args-for-docker-compose-in-coolify.md)
for more information about build args with Docker Compose and this [test
repository](https://github.com/Cryszon/coolify-build-secrets-test) for a
deployable example.

</details>
